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
    支持以下功能：
      - manageProjects 管理工程
      - manageMiniCode 管理代码片段
      - createMiniCode 创建代码片段
      - importMiniCode 导入代码片段
      - openProject 打开小程序
        ```
        w.openMiniProgram({
          appid: h,
          projectname: j,
          projectpath: k
        })
        ```
      - createProject 创建程序
        ```
        dispatch({
          type: f.PROJECT_CREATE_PROJECT_REQUEST
        })
        ```
3. MAIN_WINDOW_TYPE.CREATE_PROJECT
    输入APPID、APPName、projectpath等创建工程
      - 验证appid
      - 验证projectname
      - 验证projectpath
      - 验证是否有效小程序工程，判断依据为：是否含有app.json、project.config.json、game.json
      - 读取配置文件project.config.json
      - 根据appid获取appinfo，应该包含一些验证逻辑 （https://servicewechat.com/wxa-dev-logic/getappinfo?appid=${d})
      - 创建工程
        ```
        w = {
          isAdmin: j.is_admin,
          isTourist: !1,
          isGameTourist: this.state.isGameTourist,
          projectid: p,
          hash: r,  //根据appid_appname的组成进行hash
          appid: d,
          projectname: e,
          projectpath: a,
          libVersion: o,
          compileType: A,
          attr: _extends({}, y, {
              platform: !!j.is_platform,
              gameApp: !!j.game_app,
              platformNickname: j.platform_nickname || '',
              appName: c,
              appImageUrl: b
          })
        }
        this.props.createProjectSuccess(w, {
          initQuickStart: x,
          quickstartHost: this.state.quickstartHost
        })
        ```
      - 打开工程：this.props.openProject(w.projectid)
        ```
        w.openMiniProgram({
          appid: h,
          projectname: j,
          projectpath: k
        })
        ```
4. 打开小程序w.openMiniProgram
```
static openMiniProgram({appid: b, projectname: e, projectpath: f, options: g={}}={}) {
    if (!b || !e || !f)
        return !1;
    let h = `./html/index.html?devtype=miniprogram&devid=${+new Date}&appid=${b}&projectname=${e}&projectpath=${f}`;
    g.isTemp && (h += '&isTemp=1'),
    g.isOnline && (h += '&isOnline=1');
    let i = `mini_program_dev_window_${b}_${e}`;
    return g.simple && (i += `_${g.userInfo.openid}`,
    h += `&simple=1&${a.stringify(g.userInfo)}`),
    d.openWindow(h, {
        id: i,
        title: `${decodeURIComponent(e)} - ${nw.App.manifest.window.title}`
    }),
    c.pendingOpenWindows.add(`${b}_${e}`),
    !0
}
```
5. 打开窗口d.openWindow
```
static openWindow(a, c={}) {
    const d = function(a, c) {
        const d = {};
        for (const b of a)
            c.hasOwnProperty(b) && (d[b] = c[b]);
        return c.width || (d.width = b.SIZE.DEFAULT.WIDTH),
        c.height || (d.height = b.SIZE.DEFAULT.HEIGHT),
        c.title || (d.title = nw.App.manifest.window.title),
        c.hasOwnProperty('focus') || (d.focus = !0),
        d.new_instance = !0,
        d
    }(['id', 'title', 'icon', 'position', 'always_on_top', 'width', 'height', 'min_width', 'min_height', 'max_width', 'max_height', 'resizable', 'kiosk', 'fullscreen', 'show_in_taskbar', 'frame', 'transparent', 'new_instance'], c);
    nw.Window.open(a, d)
}
```
