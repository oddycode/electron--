# BrowserWindow

> ブラウザウィンドウを作成したり、制御したりします。

プロセス: [Main](../glossary.md#main-process)

```javascript
// メインプロセス
const {BrowserWindow} = require('electron')

// または、レンダラープロセスから `remote` を使用してください。
// const {BrowserWindow} = require('electron').remote

let win = new BrowserWindow({width: 800, height: 600})
win.on('closed', () => {
  win = null
})

// リモートURLをロード
win.loadURL('https://github.com')

// または、ローカルファイルをロード
win.loadURL(`file://${__dirname}/app/index.html`)
```

## フレームレスウィンドウ

[Frameless Window](frameless-window.md) APIを使うと、枠がないウインドウや任意の形状の透明なウインドウを作成することができます。

## ウインドウを違和感なく表示する

ウインドウにページを直接ロードすると、ユーザにはページが徐々にロードされるように見えるかもしれません。これはネイティブアプリでは良い挙動ではありません。チラつかせることなくウインドウを表示するには、さまざまな状況に応じた2つの解決策があります。

### `ready-to-show` イベントを使用する

ページのロード中、ウインドウがまだ表示されていない場合、レンダラープロセスが初めてページをレンダリングし終わったとき、`ready-to-show` イベントが発生します。 このイベントの後にウインドウを表示させれば、チラつくことはありません。

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({show: false})
win.once('ready-to-show', () => {
  win.show()
})
```

このイベントは、通常、`did-finish-load` イベントの後に発生しますが、大量のリモートリソースがあるページでは、`did-finish-load` イベントの前に発生する可能性があります。

### `backgroundColor`を設定する

複雑なアプリでは、`ready-to-show` イベントが発生するのに時間がかかり過ぎて、アプリが遅いと感じさせる可能性があります。 このような場合、ウインドウをすぐに表示し、アプリの背景に近い `backgroundColor` を使うことを推奨します。

```javascript
const {BrowserWindow} = require('electron')

let win = new BrowserWindow({backgroundColor: '#2e2c29'})
win.loadURL('https://github.com')
```

`ready-to-show` イベントを使っているアプリに対しても、アプリがよりネイティブに感じられるように `backgroundColor` を設定することが推奨されます。

## 親ウィンドウと子ウィンドウ

`parent` オプションを使用することで、子ウインドウを作成することができます。

```javascript
const {BrowserWindow} = require('electron')

let top = new BrowserWindow()
let child = new BrowserWindow({parent: top})
child.show()
top.show()
```

`child` ウインドウは、常に `top` ウインドウの前面に表示されます。

### モーダルウィンドウ

モーダルウインドウは親ウインドウを無効化する子ウインドウです。モーダルウインドウを作成するには、`parent` と `modal` オプションの両方を設定しなければなりません。

```javascript
const {BrowserWindow} = require('electron')

let child = new BrowserWindow({parent: top, modal: true, show: false})
child.loadURL('https://github.com')
child.once('ready-to-show', () => {
  child.show()
})
```

### ページの表示状態

[Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API) は、以下のように動作します。

* すべてのプラットフォームおいて、表示状態はウインドウが非表示/最小化されているかどうかをトラッキングします。
* さらに、macOSでは、表示状態はウインドウが重なり合った状態もトラッキングします。 ウインドウが別のウインドウと重なり合った (例えば、完全に覆い隠された) 場合、表示状態は、`hidden` になります。 他のプラットフォーム上では、ウインドウが最小化されるか、明示的に `win.hide()` で非表示にされた場合のみ、表示状態は `hidden` になります。
* `BrowserWindow` が `show: false` で作成された場合、ウインドウが実際には非表示であるにも関わらず、初期の表示状態は `visible` になります。
* `backgroundThrottling` が無効の場合、ウインドウを最小化したり、重ねたり、非表示にしたりしても、表示状態は `visible` のままになります。

消費電力を最小にするために、表示状態が `hidden` のとき、高負荷な操作を一時停止することを推奨します。

### プラットフォームに関する注意事項

* macOSでは、モーダルウインドウは親ウインドウに付随したシートとして表示されます。
* 親ウインドウが移動したとき、macOSでは、子ウインドウは親ウインドウに対する相対的な位置を維持しますが、WindowsとLinuxでは、子ウインドウは移動しません。
* Windowsでは、動的に親ウインドウを変更することはサポートされていません。
* Linuxでは、モーダルウインドウの型は、`dialog` に変更されます。
* Linuxでは、多くのデスクトップ環境は、モーダルウインドウを非表示にすることをサポートしていません。

## クラス: BrowserWindow

> ブラウザウィンドウを作成したり、制御したりします。

プロセス: [Main](../glossary.md#main-process)

`BrowserWindow` は [EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter) です。

`options` によって設定されたネイティブプロパティで新しい `BrowserWindow` を生成します。

### `new BrowserWindow([options])`

* `options` Object (任意) 
  * `width` Integer (任意) - ピクセル単位でのウインドウの幅。省略値は、`800` です。
  * `height` Integer (任意) - ピクセル単位でのウインドウの高さ。省略値は、`600` です。
  * `x` Integer (任意) (yが使われている場合は、**必須**) - ウインドウの画面左のオフセット。 省略値では、ウインドウが中央になるようになっています。
  * `y` Integer (任意) (xが使われている場合は、**必須**) - ウインドウの画面上のオフセット。 省略値では、ウインドウが中央になるようになっています。
  * `useContentSize` Boolean (任意) - `width` と `height` が、Webページのサイズとして使用されます。この場合、実際のウインドウのサイズは、ウインドウ枠のサイズが含まれ、若干大きくなることを意味します。 省略値は `false` です。
  * `center` Boolean (任意) - ウインドウを画面中央に表示します。
  * `minWidth` Integer (任意) - ウインドウの最小の幅。省略値は、`` です。
  * `minHeight` Integer (任意) - ウィンドウの最小の高さ。省略値は、`` です。
  * `maxWidth` Integer (任意) - ウインドウの最大の幅。省略値は、無制限です。
  * `maxHeight` Integer (任意) - ウインドウの最大の高さ。省略値は、無制限です。
  * `resizable` Boolean (任意) - ウインドウがリサイズ可能かどうか。省略値は、 `true` です。
  * `movable` Boolean (任意) - ウインドウが移動可能かどうか。これはLinuxでは実装されていません。省略値は、`true` です。
  * `minimizable` Boolean (任意) - ウインドウが最小化可能かどうか。これはLinuxでは実装されていません。省略値は、`true` です。
  * `maximizable` Boolean (任意) - ウインドウが最大化可能かどうか。これはLinuxでは実装されていません。省略値は、`true` です。
  * `closable` Boolean (任意) - ウインドウを閉じることができるかどうか。これはLinuxでは実装されていません。省略値は、`true` です。
  * `focusable` Boolean (任意) - ウインドウにフォーカスを当てることができるかどうか。 省略値は `true` です。 Windowsでは、`focusable: false` と設定することは、`skipTaskbar: true` と設定することにもなります。 Linuxでは、`focusable: false` と設定することは、ウインドウがwmとのやり取りを停止することになるため、ウインドウがすべてのワークスペースで常に前面に表示されます。
  * `alwaysOnTop` Boolean (任意) - ウインドウを常に他のウインドウの前面に表示させるかどうか。省略値は、`false` です。
  * `fullscreen` Boolean (任意) - ウインドウをフルスクリーンで表示させるかどうか。 明示的に `false` と設定された場合、macOSでは、フルスクリーンボタンが非表示または無効になります。 省略値は `false` です。
  * `fullscreenable` Boolean (任意) - ウインドウをフルスクリーンモードにすることができるかどうか。 macOSでは、さらに、最大化/ズームボタンが、フルスクリーンモードまたはウインドウ最大化に切り替わるかどうか。 省略値は `true` です。
  * `simpleFullscreen` Boolean (任意) - macOSでLionより前のフルスクリーンを使用します。省略値は、`false` です。
  * `skipTaskbar` Boolean (任意) - ウインドウをタスクバーに表示するかどうか。省略値は、`false` です。
  * `kiosk` Boolean (任意) - キオスクモード。省略値は、`false` です。
  * `title` String (任意) - 既定のウインドウタイトル。省略値は、`"Electron"` です。
  * `icon` ([NativeImage](native-image.md) | String) (任意) - ウインドウのアイコン。 Windowsでは、最高の視覚効果を得るためには、`ICO` アイコンを使うことを推奨します。未定義のままにすることもできますが、その場合、実行可能ファイルのアイコンが使われます。
  * `show` Boolean (任意) - 生成時にウインドウを表示するかどうか。省略値は、`true` です。
  * `frame` Boolean (任意) - [Frameless Window](frameless-window.md) を生成する場合、`false` に指定します。省略値は、`true` です。
  * `parent` BrowserWindow (任意) - 親ウインドウを指定します。省略値は、`null` です。
  * `modal` Boolean (任意) - モーダルウインドウかどうか。ウインドウが子ウインドウのときだけ機能します。 省略値は、`false` です。
  * `acceptFirstMouse` Boolean (任意) - ウインドウをアクティブにした単発のマウスダウンイベントを同時にWebビューが受け付けるかどうか。省略値は、`false` です。
  * `disableAutoHideCursor` Boolean (任意) - 入力中にカーソルを非表示にするかどうか。 省略値は、`false` です。
  * `autoHideMenuBar` Boolean (任意) - `Alt` キーが押されていない場合、メニューバーを自動的に非表示にします。省略値は、`false` です。
  * `enableLargerThanScreen` Boolean (任意) - ウインドウを画面よりも大きいサイズに変更できるようにします。省略値は、`false` です。
  * `backgroundColor` String (任意) - `#66CD00` や `#FFF` や `#80FFFFFF` (アルファ値をサポートしています) のような16進数の値でのウインドウの背景色。 省略値は、`#FFF` (白) です。
  * `hasShadow` Boolean (任意) - ウインドウが影をもっているかどうか。これはmacOSでのみ実装されています。省略値は、`true` です。
  * `opacity` Number (任意) - ウインドウの初期透明度を0.0 (完全に透明) から 1.0 (完全に不透明) の間で設定します。これはWindowsとmacOSでのみ実装されています。
  * `darkTheme` Boolean (任意) - ウインドウに対してダークテーマを強制的に使用します。いくつかのGTK+3デスクトップ環境でしか動作しません。省略値は、`false` です。
  * `transparent` Boolean (任意) - ウインドウを [透明](frameless-window.md) にします。 省略値は、`false` です。
  * `type` String (任意) - ウインドウのタイプで、省略値は通常のウインドウです。この詳細は以下を参照してください。
  * `titleBarStyle` String (任意) - ウインドウのタイトルバーのスタイル。 省略値は、 `default`です。取りうる値: 
    * `default` - 標準の灰色不透明なMacのタイトルバーになります。
    * `hidden` - タイトルバーが非表示かつフルサイズのコンテンツウインドウになりますが、タイトルバーには、まだ標準のウインドウコントロール ("信号") が左上にあります。
    * `hidden-inset` - 非推奨のため、代わりに、`hiddenInset` を使用してください。
    * `hiddenInset` - ウインドウの端から信号ボタンが少し埋め込まれた別の見た目でタイトルバーが非表示になります。
    * `customButtonsOnHover` Boolean (任意) - macOSのフレームレスウインドウで、カスタムの閉じる、最小化、フルスクリーンボタンを描画します。 これらのボタンはウインドウの左上を通過させないと表示されません。 これらのボタンは標準のウインドウツールバーボタンで発生するマウスイベントの問題を防止します。 **注:** このオプションは、現在、実験的なものです。
  * `fullscreenWindowTitle` Boolean (任意) - macOSのフルスクリーンモードで、どの `titleBarStyle` オプションの場合でもタイトルバーにタイトルを表示します。省略値は、`false` です。
  * `thickFrame` Boolean (任意) - Windowsのフレームレスウインドウに対して、標準のウインドウ枠を追加する `WS_THICKFRAME` スタイルを使用します。 `false` に設定すると、ウインドウの影とウインドウアニメーションがなくなります。 省略値は `true` です。
  * `vibrancy` String (任意) - macOSでのみ、ウインドウに曇りガラス効果の種類を追加します。 `appearance-based`、`light`、`dark`、`titlebar`、`selection`、`menu`、`popover`、`sidebar`、`medium-light` または `ultra-dark` にすることができます。
  * `zoomToPageWidth` Boolean (任意) - macOSで、optionキーを押下しながら緑の信号ボタンをクリックしたり、ウインドウ > ズーム のメニュー項目をクリックしたりしたときの動作を制御します。 `true` の場合、ズームしたとき、ウインドウはWebページの最適な幅に拡大されます。`false` だと、画面の幅にズームされます。 これは、`maximize()` を直接呼び出したときの動作にも影響を与えます。 省略値は `false` です。
  * `tabbingIdentifier` String (任意) - タブのグループ名で、macOS 10.12以上の場合、ネイティブのタブとしてウインドウを開くことができます。 同一のタブ識別子を持つウインドウは、一緒にグループ化されます。 これはネイティブのタブボタンをウインドウのタブバーに追加し、`app` とウインドウが `new-window-for-tab` イベントを受け取ることができるようになります。
  * `webPreferences` Object (任意) - Webページの機能設定。 
    * `devTools` Boolean (任意) - デベロッパーツールを有効にするかどうか。 `false` に設定すると、`BrowserWindow.webContents.openDevTools()` を使ってデベロッパーツールを開くことはできません。 省略値は `true` です。
    * `nodeIntegration` Boolean (任意) - Node統合を有効にするかどうか。省略値は、`true` です。
    * `nodeIntegrationInWorker` Boolean (任意) - WebワーカーでNode統合を有効にするかどうか。 省略値は `false` です。 これについての詳細は、[マルチスレッド](../tutorial/multithreading.md) を参照してください。
    * `preload` String (optional) - Specifies a script that will be loaded before other scripts run in the page. This script will always have access to node APIs no matter whether node integration is turned on or off. The value should be the absolute file path to the script. When node integration is turned off, the preload script can reintroduce Node global symbols back to the global scope. See example [here](process.md#event-loaded).
    * `sandbox` Boolean (optional) - If set, this will sandbox the renderer associated with the window, making it compatible with the Chromium OS-level sandbox and disabling the Node.js engine. This is not the same as the `nodeIntegration` option and the APIs available to the preload script are more limited. Read more about the option [here](sandbox-option.md). **Note:** This option is currently experimental and may change or be removed in future Electron releases.
    * `session` [Session](session.md#class-session) (optional) - Sets the session used by the page. Instead of passing the Session object directly, you can also choose to use the `partition` option instead, which accepts a partition string. When both `session` and `partition` are provided, `session` will be preferred. Default is the default session.
    * `partition` String (optional) - Sets the session used by the page according to the session's partition string. If `partition` starts with `persist:`, the page will use a persistent session available to all pages in the app with the same `partition`. If there is no `persist:` prefix, the page will use an in-memory session. By assigning the same `partition`, multiple pages can share the same session. Default is the default session.
    * `zoomFactor` Number (optional) - The default zoom factor of the page, `3.0` represents `300%`. Default is `1.0`.
    * `javascript` Boolean (optional) - Enables JavaScript support. Default is `true`.
    * `webSecurity` Boolean (optional) - When `false`, it will disable the same-origin policy (usually using testing websites by people), and set `allowRunningInsecureContent` to `true` if this options has not been set by user. 省略値は `true` です。
    * `allowRunningInsecureContent` Boolean (optional) - Allow an https page to run JavaScript, CSS or plugins from http URLs. Default is `false`.
    * `images` Boolean (optional) - Enables image support. Default is `true`.
    * `textAreasAreResizable` Boolean (optional) - Make TextArea elements resizable. Default is `true`.
    * `webgl` Boolean (optional) - Enables WebGL support. Default is `true`.
    * `webaudio` Boolean (optional) - Enables WebAudio support. Default is `true`.
    * `plugins` Boolean (optional) - Whether plugins should be enabled. Default is `false`.
    * `experimentalFeatures` Boolean (optional) - Enables Chromium's experimental features. Default is `false`.
    * `experimentalCanvasFeatures` Boolean (optional) - Enables Chromium's experimental canvas features. Default is `false`.
    * `scrollBounce` Boolean (optional) - Enables scroll bounce (rubber banding) effect on macOS. Default is `false`.
    * `blinkFeatures` String (optional) - A list of feature strings separated by `,`, like `CSSVariables,KeyboardEventKey` to enable. The full list of supported feature strings can be found in the [RuntimeEnabledFeatures.json5](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/platform/runtime_enabled_features.json5?l=70) file.
    * `disableBlinkFeatures` String (optional) - A list of feature strings separated by `,`, like `CSSVariables,KeyboardEventKey` to disable. The full list of supported feature strings can be found in the [RuntimeEnabledFeatures.json5](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/platform/runtime_enabled_features.json5?l=70) file.
    * `defaultFontFamily` Object (optional) - Sets the default font for the font-family. 
      * `standard` String (optional) - Defaults to `Times New Roman`.
      * `serif` String (optional) - Defaults to `Times New Roman`.
      * `sansSerif` String (optional) - Defaults to `Arial`.
      * `monospace` String (optional) - Defaults to `Courier New`.
      * `cursive` String (optional) - Defaults to `Script`.
      * `fantasy` String (optional) - Defaults to `Impact`.
    * `defaultFontSize` Integer (optional) - Defaults to `16`.
    * `defaultMonospaceFontSize` Integer (optional) - Defaults to `13`.
    * `minimumFontSize` Integer (optional) - Defaults to ``.
    * `defaultEncoding` String (optional) - Defaults to `ISO-8859-1`.
    * `backgroundThrottling` Boolean (optional) - Whether to throttle animations and timers when the page becomes background. This also affects the [Page Visibility API](#page-visibility). Defaults to `true`.
    * `offscreen` Boolean (optional) - Whether to enable offscreen rendering for the browser window. Defaults to `false`. See the [offscreen rendering tutorial](../tutorial/offscreen-rendering.md) for more details.
    * `contextIsolation` Boolean (optional) - Whether to run Electron APIs and the specified `preload` script in a separate JavaScript context. Defaults to `false`. The context that the `preload` script runs in will still have full access to the `document` and `window` globals but it will use its own set of JavaScript builtins (`Array`, `Object`, `JSON`, etc.) and will be isolated from any changes made to the global environment by the loaded page. The Electron API will only be available in the `preload` script and not the loaded page. This option should be used when loading potentially untrusted remote content to ensure the loaded content cannot tamper with the `preload` script and any Electron APIs being used. This option uses the same technique used by [Chrome Content Scripts](https://developer.chrome.com/extensions/content_scripts#execution-environment). You can access this context in the dev tools by selecting the 'Electron Isolated Context' entry in the combo box at the top of the Console tab. **Note:** This option is currently experimental and may change or be removed in future Electron releases.
    * `nativeWindowOpen` Boolean (optional) - Whether to use native `window.open()`. Defaults to `false`. **Note:** This option is currently experimental.
    * `webviewTag` Boolean (optional) - Whether to enable the [`<webview>` tag](webview-tag.md). Defaults to the value of the `nodeIntegration` option. **Note:** The `preload` script configured for the `<webview>` will have node integration enabled when it is executed so you should ensure remote/untrusted content is not able to create a `<webview>` tag with a possibly malicious `preload` script. You can use the `will-attach-webview` event on [webContents](web-contents.md) to strip away the `preload` script and to validate or alter the `<webview>`'s initial settings.

When setting minimum or maximum window size with `minWidth`/`maxWidth`/ `minHeight`/`maxHeight`, it only constrains the users. It won't prevent you from passing a size that does not follow size constraints to `setBounds`/`setSize` or to the constructor of `BrowserWindow`.

The possible values and behaviors of the `type` option are platform dependent. Possible values are:

* On Linux, possible types are `desktop`, `dock`, `toolbar`, `splash`, `notification`.
* On macOS, possible types are `desktop`, `textured`. 
  * The `textured` type adds metal gradient appearance (`NSTexturedBackgroundWindowMask`).
  * The `desktop` type places the window at the desktop background window level (`kCGDesktopWindowLevel - 1`). Note that desktop window will not receive focus, keyboard or mouse events, but you can use `globalShortcut` to receive input sparingly.
* On Windows, possible type is `toolbar`.

### インスタンスイベント

Objects created with `new BrowserWindow` emit the following events:

**Note:** Some events are only available on specific operating systems and are labeled as such.

#### イベント: 'page-title-updated'

戻り値：

* `event` Event
* `title` String

Emitted when the document changed its title, calling `event.preventDefault()` will prevent the native window's title from changing.

#### イベント: 'close'

戻り値：

* `event` Event

Emitted when the window is going to be closed. It's emitted before the `beforeunload` and `unload` event of the DOM. Calling `event.preventDefault()` will cancel the close.

Usually you would want to use the `beforeunload` handler to decide whether the window should be closed, which will also be called when the window is reloaded. In Electron, returning any value other than `undefined` would cancel the close. For example:

```javascript
window.onbeforeunload = (e) => {
  console.log('I do not want to be closed')

  // Unlike usual browsers that a message box will be prompted to users, returning
  // a non-void value will silently cancel the close.
  // It is recommended to use the dialog API to let the user confirm closing the
  // application.
  e.returnValue = false // equivalent to `return false` but not recommended
}
```

***Note**: There is a subtle difference between the behaviors of `window.onbeforeunload = handler` and `window.addEventListener('beforeunload', handler)`. It is recommended to always set the `event.returnValue` explicitly, instead of just returning a value, as the former works more consistently within Electron.*

#### イベント: 'closed'

Emitted when the window is closed. After you have received this event you should remove the reference to the window and avoid using it any more.

#### イベント: 'session-end' *Windows*

Emitted when window session is going to end due to force shutdown or machine restart or session log off.

#### イベント: 'unresponsive'

Emitted when the web page becomes unresponsive.

#### イベント: 'responsive'

Emitted when the unresponsive web page becomes responsive again.

#### イベント: 'blur'

Emitted when the window loses focus.

#### イベント: 'focus'

Emitted when the window gains focus.

#### イベント: 'show'

Emitted when the window is shown.

#### イベント: 'hide'

Emitted when the window is hidden.

#### イベント: 'ready-to-show'

Emitted when the web page has been rendered (while not being shown) and window can be displayed without a visual flash.

#### イベント: 'maximize'

Emitted when window is maximized.

#### イベント: 'unmaximize'

Emitted when the window exits from a maximized state.

#### イベント: 'minimize'

Emitted when the window is minimized.

#### イベント: 'restore'

Emitted when the window is restored from a minimized state.

#### イベント: 'resize'

Emitted when the window is being resized.

#### イベント: 'move'

Emitted when the window is being moved to a new position.

**Note**: On macOS this event is just an alias of `moved`.

#### イベント: 'moved' *macOS*

Emitted once when the window is moved to a new position.

#### イベント: 'enter-full-screen'

Emitted when the window enters a full-screen state.

#### イベント: 'leave-full-screen'

Emitted when the window leaves a full-screen state.

#### イベント: 'enter-html-full-screen'

Emitted when the window enters a full-screen state triggered by HTML API.

#### イベント: 'leave-html-full-screen'

Emitted when the window leaves a full-screen state triggered by HTML API.

#### イベント: 'app-command' *Windows*

戻り値：

* `event` Event
* `command` String

Emitted when an [App Command](https://msdn.microsoft.com/en-us/library/windows/desktop/ms646275(v=vs.85).aspx) is invoked. These are typically related to keyboard media keys or browser commands, as well as the "Back" button built into some mice on Windows.

Commands are lowercased, underscores are replaced with hyphens, and the `APPCOMMAND_` prefix is stripped off. e.g. `APPCOMMAND_BROWSER_BACKWARD` is emitted as `browser-backward`.

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()
win.on('app-command', (e, cmd) => {
  // Navigate the window back when the user hits their mouse back button
  if (cmd === 'browser-backward' && win.webContents.canGoBack()) {
    win.webContents.goBack()
  }
})
```

#### イベント: 'scroll-touch-begin' *macOS*

Emitted when scroll wheel event phase has begun.

#### イベント: 'scroll-touch-end' *macOS*

Emitted when scroll wheel event phase has ended.

#### イベント: 'scroll-touch-edge' *macOS*

Emitted when scroll wheel event phase filed upon reaching the edge of element.

#### イベント: 'swipe' *macOS*

戻り値：

* `event` Event
* `direction` String

Emitted on 3-finger swipe. Possible directions are `up`, `right`, `down`, `left`.

#### イベント: 'sheet-begin' *macOS*

Emitted when the window opens a sheet.

#### イベント: 'sheet-end' *macOS*

Emitted when the window has closed a sheet.

#### イベント: 'new-window-for-tab' *macOS*

Emitted when the native new tab button is clicked.

### 静的メソッド

The `BrowserWindow` class has the following static methods:

#### `BrowserWindow.getAllWindows()`

Returns `BrowserWindow[]` - An array of all opened browser windows.

#### `BrowserWindow.getFocusedWindow()`

Returns `BrowserWindow` - The window that is focused in this application, otherwise returns `null`.

#### `BrowserWindow.fromWebContents(webContents)`

* `webContents` [WebContents](web-contents.md)

Returns `BrowserWindow` - The window that owns the given `webContents`.

#### `BrowserWindow.fromBrowserView(browserView)`

* `browserView` [BrowserView](browser-view.md)

Returns `BrowserWindow | null` - The window that owns the given `browserView`. If the given view is not attached to any window, returns `null`.

#### `BrowserWindow.fromId(id)`

* `id` Integer

Returns `BrowserWindow` - The window with the given `id`.

#### `BrowserWindow.addExtension(path)`

* `path` String

Adds Chrome extension located at `path`, and returns extension's name.

The method will also not return if the extension's manifest is missing or incomplete.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.removeExtension(name)`

* `name` String

Remove a Chrome extension by name.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.getExtensions()`

Returns `Object` - The keys are the extension names and each value is an Object containing `name` and `version` properties.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.addDevToolsExtension(path)`

* `path` String

Adds DevTools extension located at `path`, and returns extension's name.

The extension will be remembered so you only need to call this API once, this API is not for programming use. If you try to add an extension that has already been loaded, this method will not return and instead log a warning to the console.

The method will also not return if the extension's manifest is missing or incomplete.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.removeDevToolsExtension(name)`

* `name` String

Remove a DevTools extension by name.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.getDevToolsExtensions()`

Returns `Object` - The keys are the extension names and each value is an Object containing `name` and `version` properties.

To check if a DevTools extension is installed you can run the following:

```javascript
const {BrowserWindow} = require('electron')

let installed = BrowserWindow.getDevToolsExtensions().hasOwnProperty('devtron')
console.log(installed)
```

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

### インスタンスプロパティ

Objects created with `new BrowserWindow` have the following properties:

```javascript
const {BrowserWindow} = require('electron')
// In this example `win` is our instance
let win = new BrowserWindow({width: 800, height: 600})
win.loadURL('https://github.com')
```

#### `win.webContents`

A `WebContents` object this window owns. All web page related events and operations will be done via it.

See the [`webContents` documentation](web-contents.md) for its methods and events.

#### `win.id`

A `Integer` representing the unique ID of the window.

### インスタンスメソッド

Objects created with `new BrowserWindow` have the following instance methods:

**注:** いくつかのメソッドは特定のオペレーティングシステムでのみ利用可能で、そのように注記がつけられています。

#### `win.destroy()`

Force closing the window, the `unload` and `beforeunload` event won't be emitted for the web page, and `close` event will also not be emitted for this window, but it guarantees the `closed` event will be emitted.

#### `win.close()`

Try to close the window. This has the same effect as a user manually clicking the close button of the window. The web page may cancel the close though. See the [close event](#event-close).

#### `win.focus()`

Focuses on the window.

#### `win.blur()`

Removes focus from the window.

#### `win.isFocused()`

Returns `Boolean` - Whether the window is focused.

#### `win.isDestroyed()`

Returns `Boolean` - Whether the window is destroyed.

#### `win.show()`

Shows and gives focus to the window.

#### `win.showInactive()`

Shows the window but doesn't focus on it.

#### `win.hide()`

Hides the window.

#### `win.isVisible()`

Returns `Boolean` - Whether the window is visible to the user.

#### `win.isModal()`

Returns `Boolean` - Whether current window is a modal window.

#### `win.maximize()`

Maximizes the window. This will also show (but not focus) the window if it isn't being displayed already.

#### `win.unmaximize()`

Unmaximizes the window.

#### `win.isMaximized()`

Returns `Boolean` - Whether the window is maximized.

#### `win.minimize()`

Minimizes the window. On some platforms the minimized window will be shown in the Dock.

#### `win.restore()`

Restores the window from minimized state to its previous state.

#### `win.isMinimized()`

Returns `Boolean` - Whether the window is minimized.

#### `win.setFullScreen(flag)`

* `flag` Boolean

Sets whether the window should be in fullscreen mode.

#### `win.isFullScreen()`

Returns `Boolean` - Whether the window is in fullscreen mode.

#### `win.setSimpleFullScreen(flag)` *macOS*

* `flag` Boolean

Enters or leaves simple fullscreen mode.

Simple fullscreen mode emulates the native fullscreen behavior found in versions of Mac OS X prior to Lion (10.7).

#### `win.isSimpleFullScreen()` *macOS*

Returns `Boolean` - Whether the window is in simple (pre-Lion) fullscreen mode.

#### `win.setAspectRatio(aspectRatio[, extraSize])` *macOS*

* `aspectRatio` Float - The aspect ratio to maintain for some portion of the content view.
* `extraSize` [Size](structures/size.md) - The extra size not to be included while maintaining the aspect ratio.

This will make a window maintain an aspect ratio. The extra size allows a developer to have space, specified in pixels, not included within the aspect ratio calculations. This API already takes into account the difference between a window's size and its content size.

Consider a normal window with an HD video player and associated controls. Perhaps there are 15 pixels of controls on the left edge, 25 pixels of controls on the right edge and 50 pixels of controls below the player. In order to maintain a 16:9 aspect ratio (standard aspect ratio for HD @1920x1080) within the player itself we would call this function with arguments of 16/9 and [ 40, 50 ]. The second argument doesn't care where the extra width and height are within the content view--only that they exist. Just sum any extra width and height areas you have within the overall content view.

#### `win.previewFile(path[, displayName])` *macOS*

* `path` String - The absolute path to the file to preview with QuickLook. This is important as Quick Look uses the file name and file extension on the path to determine the content type of the file to open.
* `displayName` String (optional) - The name of the file to display on the Quick Look modal view. This is purely visual and does not affect the content type of the file. Defaults to `path`.

Uses [Quick Look](https://en.wikipedia.org/wiki/Quick_Look) to preview a file at a given path.

#### `win.closeFilePreview()` *macOS*

Closes the currently open [Quick Look](https://en.wikipedia.org/wiki/Quick_Look) panel.

#### `win.setBounds(bounds[, animate])`

* `bounds` [Rectangle](structures/rectangle.md) 
* `animate` Boolean (optional) *macOS*

Resizes and moves the window to the supplied bounds

#### `win.getBounds()`

Returns [`Rectangle`](structures/rectangle.md)

#### `win.setContentBounds(bounds[, animate])`

* `bounds` [Rectangle](structures/rectangle.md) 
* `animate` Boolean (optional) *macOS*

Resizes and moves the window's client area (e.g. the web page) to the supplied bounds.

#### `win.getContentBounds()`

Returns [`Rectangle`](structures/rectangle.md)

#### `win.setSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (optional) *macOS*

Resizes the window to `width` and `height`.

#### `win.getSize()`

Returns `Integer[]` - Contains the window's width and height.

#### `win.setContentSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (optional) *macOS*

Resizes the window's client area (e.g. the web page) to `width` and `height`.

#### `win.getContentSize()`

Returns `Integer[]` - Contains the window's client area's width and height.

#### `win.setMinimumSize(width, height)`

* `width` Integer
* `height` Integer

Sets the minimum size of window to `width` and `height`.

#### `win.getMinimumSize()`

Returns `Integer[]` - Contains the window's minimum width and height.

#### `win.setMaximumSize(width, height)`

* `width` Integer
* `height` Integer

Sets the maximum size of window to `width` and `height`.

#### `win.getMaximumSize()`

Returns `Integer[]` - Contains the window's maximum width and height.

#### `win.setResizable(resizable)`

* `resizable` Boolean

Sets whether the window can be manually resized by user.

#### `win.isResizable()`

Returns `Boolean` - Whether the window can be manually resized by user.

#### `win.setMovable(movable)` *macOS* *Windows*

* `movable` Boolean

Sets whether the window can be moved by user. On Linux does nothing.

#### `win.isMovable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be moved by user.

On Linux always returns `true`.

#### `win.setMinimizable(minimizable)` *macOS* *Windows*

* `minimizable` Boolean

Sets whether the window can be manually minimized by user. On Linux does nothing.

#### `win.isMinimizable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually minimized by user

On Linux always returns `true`.

#### `win.setMaximizable(maximizable)` *macOS* *Windows*

* `maximizable` Boolean

Sets whether the window can be manually maximized by user. On Linux does nothing.

#### `win.isMaximizable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually maximized by user.

On Linux always returns `true`.

#### `win.setFullScreenable(fullscreenable)`

* `fullscreenable` Boolean

Sets whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.isFullScreenable()`

Returns `Boolean` - Whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.setClosable(closable)` *macOS* *Windows*

* `closable` Boolean

Sets whether the window can be manually closed by user. On Linux does nothing.

#### `win.isClosable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually closed by user.

On Linux always returns `true`.

#### `win.setAlwaysOnTop(flag[, level][, relativeLevel])`

* `flag` Boolean
* `level` String (optional) *macOS* - Values include `normal`, `floating`, `torn-off-menu`, `modal-panel`, `main-menu`, `status`, `pop-up-menu`, `screen-saver`, and ~~`dock`~~ (Deprecated). The default is `floating`. See the [macOS docs](https://developer.apple.com/reference/appkit/nswindow/1664726-window_levels) for more details.
* `relativeLevel` Integer (optional) *macOS* - The number of layers higher to set this window relative to the given `level`. The default is ``. Note that Apple discourages setting levels higher than 1 above `screen-saver`.

Sets whether the window should show always on top of other windows. After setting this, the window is still a normal window, not a toolbox window which can not be focused on.

#### `win.isAlwaysOnTop()`

Returns `Boolean` - Whether the window is always on top of other windows.

#### `win.center()`

Moves window to the center of the screen.

#### `win.setPosition(x, y[, animate])`

* `x` Integer
* `y` Integer
* `animate` Boolean (optional) *macOS*

Moves window to `x` and `y`.

#### `win.getPosition()`

Returns `Integer[]` - Contains the window's current position.

#### `win.setTitle(title)`

* `title` String

Changes the title of native window to `title`.

#### `win.getTitle()`

Returns `String` - The title of the native window.

**Note:** The title of web page can be different from the title of the native window.

#### `win.setSheetOffset(offsetY[, offsetX])` *macOS*

* `offsetY` Float
* `offsetX` Float (optional)

Changes the attachment point for sheets on macOS. By default, sheets are attached just below the window frame, but you may want to display them beneath a HTML-rendered toolbar. For example:

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()

let toolbarRect = document.getElementById('toolbar').getBoundingClientRect()
win.setSheetOffset(toolbarRect.height)
```

#### `win.flashFrame(flag)`

* `flag` Boolean

Starts or stops flashing the window to attract user's attention.

#### `win.setSkipTaskbar(skip)`

* `skip` Boolean

Makes the window not show in the taskbar.

#### `win.setKiosk(flag)`

* `flag` Boolean

Enters or leaves the kiosk mode.

#### `win.isKiosk()`

Returns `Boolean` - Whether the window is in kiosk mode.

#### `win.getNativeWindowHandle()`

Returns `Buffer` - The platform-specific handle of the window.

The native type of the handle is `HWND` on Windows, `NSView*` on macOS, and `Window` (`unsigned long`) on Linux.

#### `win.hookWindowMessage(message, callback)` *Windows*

* `message` Integer
* `callback` Function

Hooks a windows message. The `callback` is called when the message is received in the WndProc.

#### `win.isWindowMessageHooked(message)` *Windows*

* `message` Integer

Returns `Boolean` - `true` or `false` depending on whether the message is hooked.

#### `win.unhookWindowMessage(message)` *Windows*

* `message` Integer

Unhook the window message.

#### `win.unhookAllWindowMessages()` *Windows*

Unhooks all of the window messages.

#### `win.setRepresentedFilename(filename)` *macOS*

* `filename` String

Sets the pathname of the file the window represents, and the icon of the file will show in window's title bar.

#### `win.getRepresentedFilename()` *macOS*

Returns `String` - The pathname of the file the window represents.

#### `win.setDocumentEdited(edited)` *macOS*

* `edited` Boolean

Specifies whether the window’s document has been edited, and the icon in title bar will become gray when set to `true`.

#### `win.isDocumentEdited()` *macOS*

Returns `Boolean` - Whether the window's document has been edited.

#### `win.focusOnWebView()`

#### `win.blurWebView()`

#### `win.capturePage([rect, ]callback)`

* `rect` [Rectangle](structures/rectangle.md) (optional) - The bounds to capture
* `callback` Function 
  * `image` [NativeImage](native-image.md)

Same as `webContents.capturePage([rect, ]callback)`.

#### `win.loadURL(url[, options])`

* `url` String
* `options` Object (optional) 
  * `httpReferrer` String (optional) - A HTTP Referrer url.
  * `userAgent` String (optional) - A user agent originating the request.
  * `extraHeaders` String (optional) - Extra headers separated by "\n"
  * `postData` ([UploadRawData[]](structures/upload-raw-data.md) | [UploadFile[]](structures/upload-file.md) | [UploadFileSystem[]](structures/upload-file-system.md) | [UploadBlob[]](structures/upload-blob.md)) - (optional)
  * `baseURLForDataURL` String (optional) - Base url (with trailing path separator) for files to be loaded by the data url. This is needed only if the specified `url` is a data url and needs to load other files.

Same as `webContents.loadURL(url[, options])`.

The `url` can be a remote address (e.g. `http://`) or a path to a local HTML file using the `file://` protocol.

To ensure that file URLs are properly formatted, it is recommended to use Node's [`url.format`](https://nodejs.org/api/url.html#url_url_format_urlobject) method:

```javascript
let url = require('url').format({
  protocol: 'file',
  slashes: true,
  pathname: require('path').join(__dirname, 'index.html')
})

win.loadURL(url)
```

You can load a URL using a `POST` request with URL-encoded data by doing the following:

```javascript
win.loadURL('http://localhost:8000/post', {
  postData: [{
    type: 'rawData',
    bytes: Buffer.from('hello=world')
  }],
  extraHeaders: 'Content-Type: application/x-www-form-urlencoded'
})
```

#### `win.reload()`

Same as `webContents.reload`.

#### `win.setMenu(menu)` *Linux* *Windows*

* `menu` Menu | null

Sets the `menu` as the window's menu bar, setting it to `null` will remove the menu bar.

#### `win.setProgressBar(progress[, options])`

* `progress` Double
* `options` Object (optional) 
  * `mode` String *Windows* - Mode for the progress bar. Can be `none`, `normal`, `indeterminate`, `error`, or `paused`.

Sets progress value in progress bar. Valid range is [0, 1.0].

Remove progress bar when progress < 0; Change to indeterminate mode when progress > 1.

On Linux platform, only supports Unity desktop environment, you need to specify the `*.desktop` file name to `desktopName` field in `package.json`. By default, it will assume `app.getName().desktop`.

On Windows, a mode can be passed. Accepted values are `none`, `normal`, `indeterminate`, `error`, and `paused`. If you call `setProgressBar` without a mode set (but with a value within the valid range), `normal` will be assumed.

#### `win.setOverlayIcon(overlay, description)` *Windows*

* `overlay` [NativeImage](native-image.md) - the icon to display on the bottom right corner of the taskbar icon. If this parameter is `null`, the overlay is cleared
* `description` String - a description that will be provided to Accessibility screen readers

Sets a 16 x 16 pixel overlay onto the current taskbar icon, usually used to convey some sort of application status or to passively notify the user.

#### `win.setHasShadow(hasShadow)` *macOS*

* `hasShadow` Boolean

Sets whether the window should have a shadow. On Windows and Linux does nothing.

#### `win.hasShadow()` *macOS*

Returns `Boolean` - Whether the window has a shadow.

On Windows and Linux always returns `true`.

#### `win.setOpacity(opacity)` *Windows* *macOS*

* `opacity` Number - between 0.0 (fully transparent) and 1.0 (fully opaque)

Sets the opacity of the window. On Linux does nothing.

#### `win.getOpacity()` *Windows* *macOS*

Returns `Number` - between 0.0 (fully transparent) and 1.0 (fully opaque)

#### `win.setThumbarButtons(buttons)` *Windows*

* `buttons` [ThumbarButton[]](structures/thumbar-button.md)

Returns `Boolean` - Whether the buttons were added successfully

Add a thumbnail toolbar with a specified set of buttons to the thumbnail image of a window in a taskbar button layout. Returns a `Boolean` object indicates whether the thumbnail has been added successfully.

The number of buttons in thumbnail toolbar should be no greater than 7 due to the limited room. Once you setup the thumbnail toolbar, the toolbar cannot be removed due to the platform's limitation. But you can call the API with an empty array to clean the buttons.

The `buttons` is an array of `Button` objects:

* `Button` Object 
  * `icon` [NativeImage](native-image.md) - ツールバーサムネイルで表示されるアイコン。 
  * `click` Function
  * `tooltip` String (optional) - ボタンのツールチップのテキスト。
  * `flags` String[] (optional) - 特定の状態や行動を制御する ボタン。デフォルトでは、 `['enabled']`。

`flags` は、 `String` を含むことができる配列です。

* `enabled` - ボタンはアクティブで、ユーザーが使用できます。
* `disabled` - ボタンは無効です。存在しますが、ユーザ操作に応答しないことを示す視覚的な状態です。
* `dismissonclick` - ボタンをクリックすると、サムネイルウインドウはすぐに閉じます。
* `nobackground` - ボタンの境界を描画しません。画像だけでしか使用しないでください。
* `hidden` - ボタンはユーザに表示されません。
* `noninteractive` - ボタンは有効ですが、反応せず、押されたボタンの状態は描画されません。この値は、通知に使われるボタンのインスタンスのために用意されてます。

#### `win.setThumbnailClip(region)` *Windows*

* `region` [Rectangle](structures/rectangle.md) - Region of the window

Sets the region of the window to show as the thumbnail image displayed when hovering over the window in the taskbar. You can reset the thumbnail to be the entire window by specifying an empty region: `{x: 0, y: 0, width: 0, height: 0}`.

#### `win.setThumbnailToolTip(toolTip)` *Windows*

* `toolTip` String

Sets the toolTip that is displayed when hovering over the window thumbnail in the taskbar.

#### `win.setAppDetails(options)` *Windows*

* `options` Object 
  * `appId` String (optional) - Window's [App User Model ID](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391569(v=vs.85).aspx). It has to be set, otherwise the other options will have no effect.
  * `appIconPath` String (optional) - Window's [Relaunch Icon](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391573(v=vs.85).aspx).
  * `appIconIndex` Integer (optional) - Index of the icon in `appIconPath`. Ignored when `appIconPath` is not set. Default is ``.
  * `relaunchCommand` String (optional) - Window's [Relaunch Command](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391571(v=vs.85).aspx).
  * `relaunchDisplayName` String (optional) - Window's [Relaunch Display Name](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391572(v=vs.85).aspx).

Sets the properties for the window's taskbar button.

**Note:** `relaunchCommand` and `relaunchDisplayName` must always be set together. If one of those properties is not set, then neither will be used.

#### `win.showDefinitionForSelection()` *macOS*

Same as `webContents.showDefinitionForSelection()`.

#### `win.setIcon(icon)` *Windows* *Linux*

* `icon` [NativeImage](native-image.md)

Changes window icon.

#### `win.setAutoHideMenuBar(hide)`

* `hide` Boolean

Sets whether the window menu bar should hide itself automatically. Once set the menu bar will only show when users press the single `Alt` key.

If the menu bar is already visible, calling `setAutoHideMenuBar(true)` won't hide it immediately.

#### `win.isMenuBarAutoHide()`

Returns `Boolean` - Whether menu bar automatically hides itself.

#### `win.setMenuBarVisibility(visible)` *Windows* *Linux*

* `visible` Boolean

Sets whether the menu bar should be visible. If the menu bar is auto-hide, users can still bring up the menu bar by pressing the single `Alt` key.

#### `win.isMenuBarVisible()`

Returns `Boolean` - Whether the menu bar is visible.

#### `win.setVisibleOnAllWorkspaces(visible)`

* `visible` Boolean

Sets whether the window should be visible on all workspaces.

**Note:** This API does nothing on Windows.

#### `win.isVisibleOnAllWorkspaces()`

Returns `Boolean` - Whether the window is visible on all workspaces.

**Note:** This API always returns false on Windows.

#### `win.setIgnoreMouseEvents(ignore[, options])`

* `ignore` Boolean
* `options` Object (optional) 
  * `forward` Boolean (optional) *Windows* - If true, forwards mouse move messages to Chromium, enabling mouse related events such as `mouseleave`. Only used when `ignore` is true. If `ignore` is false, forwarding is always disabled regardless of this value.

Makes the window ignore all mouse events.

All mouse events happened in this window will be passed to the window below this window, but if this window has focus, it will still receive keyboard events.

#### `win.setContentProtection(enable)` *macOS* *Windows*

* `enable` Boolean

Prevents the window contents from being captured by other apps.

On macOS it sets the NSWindow's sharingType to NSWindowSharingNone. On Windows it calls SetWindowDisplayAffinity with `WDA_MONITOR`.

#### `win.setFocusable(focusable)` *Windows*

* `focusable` Boolean

Changes whether the window can be focused.

#### `win.setParentWindow(parent)` *Linux* *macOS*

* `parent` BrowserWindow

Sets `parent` as current window's parent window, passing `null` will turn current window into a top-level window.

#### `win.getParentWindow()`

Returns `BrowserWindow` - The parent window.

#### `win.getChildWindows()`

Returns `BrowserWindow[]` - All child windows.

#### `win.setAutoHideCursor(autoHide)` *macOS*

* `autoHide` Boolean

Controls whether to hide cursor when typing.

#### `win.selectPreviousTab()` *macOS*

Selects the previous tab when native tabs are enabled and there are other tabs in the window.

#### `win.selectNextTab()` *macOS*

Selects the next tab when native tabs are enabled and there are other tabs in the window.

#### `win.mergeAllWindows()` *macOS*

Merges all windows into one window with multiple tabs when native tabs are enabled and there is more than one open window.

#### `win.moveTabToNewWindow()` *macOS*

Moves the current tab into a new window if native tabs are enabled and there is more than one tab in the current window.

#### `win.toggleTabBar()` *macOS*

Toggles the visibility of the tab bar if native tabs are enabled and there is only one tab in the current window.

#### `win.addTabbedWindow(browserWindow)` *macOS*

* `browserWindow` BrowserWindow

Adds a window as a tab on this window, after the tab for the window instance.

#### `win.setVibrancy(type)` *macOS*

* `type` String - Can be `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light` or `ultra-dark`. See the [macOS documentation](https://developer.apple.com/reference/appkit/nsvisualeffectview?language=objc) for more details.

Adds a vibrancy effect to the browser window. Passing `null` or an empty string will remove the vibrancy effect on the window.

#### `win.setTouchBar(touchBar)` *macOS* *Experimental*

* `touchBar` TouchBar

Sets the touchBar layout for the current window. Specifying `null` or `undefined` clears the touch bar. This method only has an effect if the machine has a touch bar and is running on macOS 10.12.1+.

**Note:** TouchBar API は現在実験的な機能です。将来的には変更されたり削除されたりする可能性があります。

#### `win.setBrowserView(browserView)` *Experimental*

* `browserView` [BrowserView](browser-view.md)

#### `win.getBrowserView()` (*実験的*)

Returns `BrowserView | null` - an attached BrowserView. Returns `null` if none is attached.

**Note:**BrowserView API は現在実験的な機能です。将来的に変更されたり削除されたりする可能性があります。