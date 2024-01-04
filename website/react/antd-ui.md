### Ant Design的Modal组件支持全屏处理

[参考博客地址](https://blog.csdn.net/m0_58016522/article/details/125039787)

```jsx
<Modal
  style={{
    maxWidth: "100vw",
    top: 0,
    paddingBottom: 0,
  }}
  bodyStyle={{
    height: "calc(100vh - 55px - 53px)",
    overflowY: "auto",
  }}
  title="Basic Modal"
  width="100vw"
  visible={isModalVisible}
  onOk={handleOk}
  onCancel={handleCancel}
>
  <p>Some contents...</p>
</Modal>
```

