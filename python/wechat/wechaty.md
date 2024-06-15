# wechaty

https://github.com/wechaty/wechaty

参考博客使用：https://blog.csdn.net/qq_47452807/article/details/137378549

### 安装

```shell
npm install qrcode-terminal --save # 支持二维码在控制台中展示
npm install wechaty  # 使用网页版微信
npm install wechaty-puppet-wechat --save  #这个依赖是关键
export WECHATY_PUPPET=wechaty-puppet-wechat #这里也是关键，需要配置你使用的puppet
```

## 实现demo

首先，要建一个聊天机器人。Wechaty 会启动一个 Puppeteer 并打开网页版微信，然后使用手机上的微信 APP 扫描 Wechaty 提供的二维码即可完成登陆。登陆成功之后，我们就可以通过 Wechaty 进行各种操作了。

```js
const qrTerm = require('qrcode-terminal');
const Wechaty = require('wechaty');

const { ScanStatus, WechatyBuilder, log } = Wechaty;

function onScan (qrcode, status) {
  if (status === ScanStatus.Waiting || status === ScanStatus.Timeout) {
    qrTerm.generate(qrcode, { small: true });  // show qrcode on console

    const qrcodeImageUrl = [
      'https://wechaty.js.org/qrcode/',
      encodeURIComponent(qrcode),
    ].join('');

    log.info('StarterBot', 'onScan: %s(%s) - %s', ScanStatus[status], status, qrcodeImageUrl);
  } else {
    log.info('StarterBot', 'onScan: %s(%s)', ScanStatus[status], status);
  }
}

// get a Wechaty instance
const bot = WechatyBuilder.build({
  name: 'wechat-bot',
  puppet: 'wechaty-puppet-wechat',
})

// emit when the bot needs to show you a QR Code for scanning
bot.on('scan', onScan);

// start the bot
bot.start()
  .then(() => log.info('StarterBot', 'Starter Bot Started.'))
  .catch(e => log.error('StarterBot', e))
```

然后，实现消息发送功能。在微信聊天机器人成功启动之后，可以用机器人发送消息，代码如下：

```js
// find a room
const room = await bot.Room.find({ topic: 'yours-wechat-group-name' })
if (room) {
  // send a message
  await room.say(`hello world`)
}
```

不难发现，使用 Wechaty 实现微信聊天机器人是很方便的，但要注意以下两点：

- Webchaty 需要 Node.js 的版本在 16+ 版本，如果低于 16 版本的话会报错。
- 相关 npm 包下载缓慢时，可以加如下设置：

```bash
npm config set puppeteer_download_host "https://npm.taobao.org/mirrors"
npm config set sharp_binary_host "https://npm.taobao.org/mirrors/sharp"
npm config set sharp_libvips_binary_host "https://npm.taobao.org/mirrors/sharp-libvips" 
```



```typescript
import { Wechaty }  from 'wechaty';

const name = 'wechat-puppet-wechat';
let bot = '';
bot = new Wechaty({
  name, // generate xxxx.memory-card.json and save login data for the next login
});

//  二维码生成
function onScan(qrcode, status) {
  require('qrcode-terminal').generate(qrcode); // 在console端显示二维码
  const qrcodeImageUrl = [
    'https://wechaty.js.org/qrcode/',
    encodeURIComponent(qrcode),
  ].join('');
  console.log(qrcodeImageUrl);
}

// 登录
async function onLogin(user) {
  console.log(`贴心小助理${user}登录了`);
  if (config.AUTOREPLY) {
    console.log(`已开启机器人自动聊天模式`);
  }
  // 登陆后创建定时任务
  await initDay();
}

//登出
function onLogout(user) {
  console.log(`小助手${user} 已经登出`);
}

bot.on('scan', onScan);
bot.on('login', onLogin);
bot.on('logout', onLogout);
bot
  .start()
  .then(() => console.log('开始登陆微信'))
  .catch((e) => console.error(e));
```

