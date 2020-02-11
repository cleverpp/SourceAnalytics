# 工程入口(React.Component)
<img src="https://github.com/cleverpp/SourceAnalytics/blob/master/wechatwebdevtools/assets/ENTRANCE.png" width="20%" height="20%" alt="ENTRANCE">

1. 选择开发类型
当前版本支持小程序(MINI_PROGRAM)、公众号网页(MP_WEB)
```
selectDevType: (a)=>(b,c)=>{
  if (a !== l.DEV_TYPE.MP_WEB)
    try {
      const a = c().project.list;
      0 < Object.keys(a).length ? b(m.setMainWindow(l.MAIN_WINDOW_TYPE.SELECT_PROJECT)) : b(m.setMainWindow(l.MAIN_WINDOW_TYPE.CREATE_PROJECT))
    } catch (a) {
      b(m.setMainWindow(l.MAIN_WINDOW_TYPE.SELECT_PROJECT))
  } else {
      w.openWebDebugger(),
      n.lastSelect = l.MAIN_WINDOW_TYPE.WEB_DEBUGGER,
      p('url_open')
  }
}
```
开发模式是小程序时，如果有过项目记录则窗口类型设置为MAIN_WINDOW_TYPE.SELECT_PROJECT，否则为MAIN_WINDOW_TYPE.CREATE_PROJECT

2. MAIN_WINDOW_TYPE.SELECT_PROJECT
    - d9315b916750758dab0e8d5a8ad99c68.js
    - 3_开发者界面_SELECT_PROJECT
3. MAIN_WINDOW_TYPE.CREATE_PROJECT
    - 307ad3efda2e8a1e598cb73a16972c84.js

