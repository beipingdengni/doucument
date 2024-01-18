## 安装

```shell
npm install @rematch/core
```

### 第一步：Init 

> **init** 用来配置你的 reducers, devtools & store
>
> index.js

```jsx
import { init } from '@rematch/core'
import * as models from './models'

const store = init({
  models,
})

export default store
```

### 第二步：Models

> 该model促使state， reducers， async actions 和 action creators 放在同一个地方。
>
> models.js

```jsx
export const count = {
  state: 0, // initial state
  reducers: {
    // handle state changes with pure functions
    increment(state, payload) {
      return state + payload
    }
  },
  effects: {
    // handle state changes with impure functions.
    // use async/await for async actions
    async incrementAsync(payload, rootState) {
      await new Promise(resolve => setTimeout(resolve, 1000))
      this.increment(payload)
    }
  }
}
```

### Step 3: Dispatch

> **dispatch** 是我们如何在你的model中触发 reducers 和 effects。 Dispatch 标准化了你的action，而无需编写action types 或者  action creators。

```jsx
import { dispatch } from '@rematch/core'

                                                  // state = { count: 0 }
// reducers
dispatch({ type: 'count/increment', payload: 1 }) // state = { count: 1 }
dispatch.count.increment(1)                       // state = { count: 2 }

// effects
dispatch({ type: 'count/incrementAsync', payload: 1 }) // state = { count: 3 } after delay
dispatch.count.incrementAsync(1)                       // state = { count: 4 } after delay
```

### Step 4: View

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider, connect } from 'react-redux'
import store from './index'

const Count = props => (
  <div>
    The count is {props.count}
    <button onClick={props.increment}>increment</button>
    <button onClick={props.incrementAsync}>incrementAsync</button>
  </div>
)

const mapState = state => ({
  count: state.count
})

const mapDispatch = ({ count: { increment, incrementAsync }}) => ({
  increment: () => increment(1),
  incrementAsync: () => incrementAsync(1)
})

const CountContainer = connect(mapState, mapDispatch)(Count)

ReactDOM.render(
  <Provider store={store}>
    <CountContainer />
  </Provider>,
  document.getElementById('root')
)
```







## TS编写参考

src/store.ts

```jsx
import { init, RematchDispatch, RematchRootState } from "@rematch/core";
import immerPlugin from "@rematch/immer";
import loadingPlugin, { ExtraModelsFromLoading } from "@rematch/loading";
import { enableMapSet } from "immer";

import models, { RootModel } from "./models";

enableMapSet();

type FullModel = ExtraModelsFromLoading<RootModel>;

const store = init<RootModel, FullModel>({
    models,
    plugins: [loadingPlugin(), immerPlugin()],
});

export default store;
export type Store = typeof store;
export type Dispatch = RematchDispatch<RootModel>;
export type RootState = RematchRootState<RootModel, FullModel>;

export const dispatch = store.dispatch;
```

src/models/index.ts

```jsx
import { Models } from "@rematch/core";

import basic, { BasicState } from "./basic";
import im, { ImState } from "./im";
import incoming, { IncomingState } from "./incoming";

interface BaseModel<T> {
    name?: string;
    state: T;
    reducers: any;
    baseReducer: any;
    effects: any;
}
export interface RootModel extends Models<RootModel> {
    basic: BaseModel<BasicState>;
    im: BaseModel<ImState>;
    incoming: BaseModel<IncomingState>;
}

export default {
    basic,
    im,
    incoming,
} as RootModel;
```

src/models/im.ts

```tsx
import { createModel } from "@rematch/core";
import { RootModel } from "./index";
import { IClientUserInfo, IWorkbenchUserInfo } from "@/types/common";
import {
    RelationInfo,
    INoticeContentMsg,
    IWorkbenchSendMsgDesc,
    WorkbenchSendMsg,
    WorkbenchSendCmd,
    WorkbenchSendStatusTypeEnum,
    IAssistantMessag,
    IAssistantInfoMap,
} from "@/types/im";
import * as effects from "./effects";
import * as imEffects from "./im-effects";
import {
    DialogEventStatusEnum,
    IBindDialogEventReq,
    IWaiterStatusProps,
    LockStatusEnum,
    WaiterStatusEnum,
} from "@/types/dialog";

export interface ImState {
    /**
     * 当前会话为前端概念
     * 可能存在的情况
     * 1. 有relationId，但实际上会话已经结束。也就是relationId有值，但没有真实的存活会话或者历史会话
     * 2. relationId可能为relationId(存活会话)，也可能为dialogId(历史会话)
     * 3. relationId与url中的:id参数保持一致
     */
    relationId: RelationIdType;
    /** 当前存活会话列表 */
    relationMap: Map<RelationIdType, RelationInfo>;
    /** 当前历史会话列表 */
    relationHistoryMap: Map<RelationIdType, RelationInfo>;
    /** 未读消息集合 */
    unreadMap: Map<RelationIdType, INoticeContentMsg>;
    /**
     * 当前所有的历史消息缓存，用于快速恢复 <RelationIdType, WorkbenchSendMsg[]>
     * 如果切换一个会话，需要判定是否为历史会话，历史会话则直接切换数据，提取数据到缓冲区
     * 如果判定为存活会话，则需要根据上下文决定是否请求一次历史会话，用于缓冲区头部插入
     * 其余的实时消息来了之后刷入缓冲区，切换之前将缓冲区内容压入此map
     */
    historyMsgMap: Map<RelationIdType, Array<WorkbenchSendMsg>>;
}

export default createModel<RootModel>()({
    state: {
        workbenchUserInfo: {
            serviceStatus: WaiterStatusEnum.Suspend,
            isWsOnline: false,
        },
        relationId: "",
        relationMap: new Map(),
        relationHistoryMap: new Map(),
    } as ImState,
    reducers: {
        setLastMsg: (state: ImState, payload: { relationId: string; lastMsg: RelationInfo["lastMsg"] }) => {
            const { relationId, lastMsg } = payload;

            const currentRelation = state.unreadMap.get(relationId) || {};

            state.unreadMap.set(relationId, {
                ...currentRelation,
                lastMsg,
            } as INoticeContentMsg);

            return state;
        },
        setServiceWelcomeText: (state: ImState, payload: string) => {
            state.workbenchUserInfo.serviceWelcomeSettings = payload;
            return state;
        },
    },
    effects: (/** dispatch */) => ({
        newDialogInfo: effects.newDialogInfo,
        sendText: imEffects.sendText,
    }),
});
```

src/models/im-effects/sendText.ts

```tsx
import { dispatch } from "@/store";
import { IChatMsg } from "@/types/im";

/** 发送文本消息 */
export default function (payload: string | Partial<IChatMsg>, state) {
    dispatch({
        type: "im/sendTextToTaget",
        payload: {
            relationId: state.im.relationId,
            msg: payload,
        },
    });
}


/** 聊天消息类型 */
export interface IChatMsg {
    /** 公司ID */
    companyId?: string;
    /** 租户ID */
    tenantId?: string;
    /** 产品线 用于区分业务功能 */
    productLine?: string;
    /** 聊天关系ID  */
    relationId: string;
    /** 消息ID 前端生成用于返回确认 */
    sendMsgId: string;
    /** 消息ID 后端生成,用于逻辑处理 */
    msgId: string;
    /** 消息发送者userId */
    fromUserId: string;
    /** 消息聊天关系类型 聊天关系类型 PRIVATE  1 单聊/  GROUP 2 群聊 */
    relationType: RelationTypeEnum;
    /** 消息标签类型 消息发送对象  */
    targetType: RelationMsgTargetTypeEnum;
    /** 消息标签值 消息发送的对象(targetType)值 TO_USER 就是toUser的Id,TO_GROUP 就是群id */
    targetValue: string[];
    /** 消息发送时间 发送时间后端生成 */
    sendTime: number | string;
    /** 操作状态，正常|撤回 */
    operateStatus: string;
    /** 消息内容类型 */
    contentType: MsgContentTypeEnum;
    /** 消息内容。需要根据contentType进行解析 */
    content: any;
    /** 提及对象类型 ALL/ 1/ 全群组  SPECIFIC/ 2/ 指定人 */
    mentionType?: string;
    /** 提及对象列表 */
    metnionList?: string[];
    /** 引用消息的id */
    referMsgId?: string;
    /** 引用消息内容 */
    referMsgContent: string;
    /** 消息撤回 */
    cmdType?: CommandEnum;
}
```

