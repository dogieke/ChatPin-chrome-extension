# ChatPin-chrome-extension

Chrome extension to easily bookmark and navigate important text snippets in LLM conversations.  
为LLM对话打造，轻松标记，快速跳转

---

# ChatPin:  LLM 对话书签

**专为大型语言模型 (LLM) 对话场景设计的 Chrome 扩展程序，旨在帮助用户轻松标记、导航与 AI (如 ChatGPT, Gemini, DeepSeek 等) 对话中的重要文本片段。**

在冗长或复杂的 LLM 对话中，我们常常会遇到一些关键信息、重要指令、有趣观点或需要稍后回顾的内容。ChatPin 允许您像在书中做标记一样，在这些文本片段旁边“固定”一个图钉，从而快速回顾和跳转。

## 核心功能

* **轻松标记文本**:
    在任何网页上 (特别适用于 LLM 对话界面)，选中您想要标记的文本，旁边会出现一个“📌”图标。点击即可在该位置创建一个书签。

* **会话级书签**:
    * 所有书签都是**临时性**的，仅在当前页面会话中有效。
    * 当标签页关闭或浏览器关闭时，该页面的所有 ChatPin 标记会自动清除。下次打开相同页面时，不会保留之前的标记。

* **两种标记模式**:
    * **多图钉模式 (Multi-pin Mode)**: (默认) 允许您在单个页面上创建多个书签。每个书签都会有一个数字编号，方便区分和导航。
    * **单图钉模式 (Single-pin Mode)**: 允许您在单个页面上只保留一个活动的图钉。创建新的单图钉会自动替换旧的。

* **模式切换与记忆**:
    * 用户可以通过扩展的弹出窗口轻松切换“多图钉”或“单图钉”模式。
    * ChatPin 会**全局记住**用户最后选择的模式（存储在 `chrome.storage.local` 中），并在所有新打开的页面上默认应用该模式。

* **弹出窗口管理 (Popup Interface)**:
    * 通过点击扩展图标或使用快捷键 (默认为 `Command+M` 或 `Ctrl+M`) 打开弹出窗口。
    * 显示当前页面所有活动书签的列表。
    * 书签列表会显示预览文本（多图钉模式下为原文前几个词，单图钉模式下为原文前几个词，具体逻辑见 `content.js` 中 `snippet` 的生成）。
    * 点击列表中的书签项，可以快速跳转到页面上对应的标记位置。
    * 在弹出窗口中切换标记模式。

* **快捷键支持**:
    * **打开/关闭 Popup**: `Command+M` (Mac) / `Ctrl+M` (Windows/Linux) - *用户可能需要在 `chrome://extensions/shortcuts` 中手动确认或设置*。
    * **单图钉模式快速跳转**: `Command+B` (Mac) / `Ctrl+B` (Windows/Linux)。
    * **多图钉模式快速跳转 (按编号)**: `Alt+1`, `Alt+2` (支持最多两个编号的快捷跳转，更多可通过 Popup 列表导航)。

* **图钉交互**:
    页面上插入的图钉图标可以直接点击以删除对应的书签。

* **SPA (单页应用) 兼容**:
    ChatPin 能够检测并适应在单页应用 (如一些现代 LLM 对话网站) 中的页面导航 (通过监听 `popstate`, `pushState`, `replaceState`)，确保书签管理与当前显示的“虚拟页面”保持同步。

* **轻量化与用户友好**:
    简洁的界面和直观的操作，专注于核心的书签功能，不干扰用户的主要浏览体验。

## 技术栈与实现细节

* **Manifest V3**:
    遵循最新的 Chrome 扩展规范。

* **Content Scripts (`content.js`)**:
    核心逻辑所在，负责：
    * 监听用户文本选择事件。
    * 在选中文本旁显示创建书签的“📌”选项。
    * 在页面上插入和管理图钉 DOM 元素。
    * 处理书签的创建、删除、编号（多图钉模式）。
    * `BookmarkManager` 类封装了主要的书签管理和页面交互逻辑。
    * 处理与 Popup 和背景脚本的消息通信。
    * 实现了对 SPA 导航的监听和适配 (`popstate`, `pushState`, `replaceState`)，确保 `BookmarkManager` 实例与当前 URL 对应的 `pageKeySuffix` 同步。

* **Popup (`popup.html`, `popup.js`)**:
    * 提供用户界面，用于显示书签列表和切换模式。
    * 通过 `chrome.tabs.sendMessage` 与 `content.js` 通信，获取书签数据和同步模式设置。
    * 使用 `chrome.storage.local` 存储全局的图钉模式偏好。

* **Background Scripts (`background.js`)**:
    * 处理 Chrome 命令 (快捷键) 的监听和分发。
    * 当中继快捷键指令到 `content.js` 时，会传递相应的操作类型（如导航到特定模式的书签）。



## 如何使用

1.  下载ChatPin zip 手动加载解压后的扩展。
2.  打开一个网页，例如一个 LLM 对话页面。
3.  选中一段您想要标记的文本。
4.  点击选区右侧出现的“📌”小图标，即可创建一个书签。
5.  点击浏览器工具栏上的 ChatPin 图标，或使用快捷键 `Command+M` / `Ctrl+M` 打开弹出窗口。
    * 您会看到当前页面已创建的书签列表。
    * 您可以选择“多图钉”或“单图钉”模式。
6.  点击 Popup 中的书签项，页面会自动滚动到对应的标记位置。
7.  使用快捷键快速导航到书签。
8.  直接点击页面上的图钉图标可以删除该书签。

## 未来可能的改进方向

* 支持跨页面/跨会话的持久化书签选项（作为一种可选模式）。
* 书签导出/导入功能。
* 更丰富的预览文本定制选项。
* 为图钉添加备注/标签功能。
* 云同步功能。

我们希望 ChatPin 能成为您在浏览和使用 LLM 服务时的得力助手！
---
# ChatPin-chrome-extension

Chrome extension to easily bookmark and navigate important text snippets in LLM conversations.  
Designed for LLM dialogues: mark with ease, navigate with speed.

---

# ChatPin: LLM Conversation Bookmarks

**ChatPin (formerly LLM Smart Bookmarks) is a Chrome extension designed specifically for Large Language Model (LLM) conversation scenarios, aiming to help users easily mark, navigate, and manage important text snippets within their dialogues with AI (such as ChatGPT, Gemini, DeepSeek, etc.).**

In lengthy or complex LLM conversations, we often encounter key information, important instructions, interesting insights, or content that needs to be revisited later. ChatPin allows you to "pin" a bookmark next to these text snippets, much like making a mark in a book, enabling quick review and navigation.

## Core Features

* **Effortless Text Marking**:
    On any webpage (especially suitable for LLM conversation interfaces), select the text you want to mark, and a “📌” icon will appear nearby. Click it to create a bookmark at that location.

* **Session-level Bookmarks**:
    * All bookmarks are **temporary** and are only valid within the current page session.
    * When the tab or browser is closed, all ChatPin marks for that page are automatically cleared. Previous marks will not be retained when the same page is reopened.

* **Two Marking Modes**:
    * **Multi-pin Mode**: (Default) Allows you to create multiple bookmarks on a single page. Each bookmark will have a numerical identifier for easy distinction and navigation.
    * **Single-pin Mode**: Allows you to keep only one active pin on a single page. Creating a new single pin automatically replaces the old one.

* **Mode Switching and Memory**:
    * Users can easily switch between "Multi-pin" or "Single-pin" mode via the extension's popup window.
    * ChatPin will **globally remember** the user's last selected mode (stored in `chrome.storage.local`) and apply it by default to all newly opened pages.

* **Popup Interface Management**:
    * Open the popup window by clicking the extension icon or using the shortcut (default: `Command+M` or `Ctrl+M`).
    * Displays a list of all active bookmarks on the current page.
    * The bookmark list shows preview text (the first few words of the original text for multi-pin mode, and the first few words for single-pin mode, with specific logic in `content.js` for snippet generation).
    * Clicking a bookmark item in the list allows you to quickly jump to the corresponding marked position on the page.
    * Switch marking modes within the popup window.

* **Shortcut Support**:
    * **Open/Close Popup**: `Command+M` (Mac) / `Ctrl+M` (Windows/Linux) - *Users may need to manually confirm or set this in `chrome://extensions/shortcuts`*.
    * **Jump to Single-pin**: `Command+B` (Mac) / `Ctrl+B` (Windows/Linux).
    * **Jump to Multi-pin (by number)**: `Alt+1`, `Alt+2` (supports shortcut navigation for up to two numbered pins; more can be navigated via the Popup list).

* **Pin Interaction**:
    Pinned icons inserted on the page can be clicked directly to delete the corresponding bookmark.

* **SPA (Single Page Application) Compatibility**:
    ChatPin can detect and adapt to page navigation within SPAs (like some modern LLM conversation websites) by listening to `popstate`, `pushState`, and `replaceState`, ensuring bookmark management stays in sync with the currently displayed "virtual page."

* **Lightweight and User-Friendly**:
    A clean interface and intuitive operation, focusing on core bookmarking functionality without interfering with the user's main Browse experience.

## Tech Stack & Implementation Details

* **Manifest V3**:
    Adheres to the latest Chrome extension specifications.

* **Content Scripts (`content.js`)**:
    The core logic resides here, responsible for:
    * Listening for user text selection events.
    * Displaying the “📌” option next to selected text to create a bookmark.
    * Inserting and managing pin DOM elements on the page.
    * Handling the creation, deletion, and numbering (in multi-pin mode) of bookmarks.
    * The `BookmarkManager` class encapsulates the main bookmark management and page interaction logic.
    * Handling message communication with the Popup and Background scripts.
    * Implementing listeners and adaptation for SPA navigation (`popstate`, `pushState`, `replaceState`) to ensure the `BookmarkManager` instance stays in sync with the `pageKeySuffix` corresponding to the current URL.

* **Popup (`popup.html`, `popup.js`)**:
    * Provides the user interface for displaying the bookmark list and switching modes.
    * Communicates with `content.js` via `chrome.tabs.sendMessage` to fetch bookmark data and synchronize mode settings.
    * Uses `chrome.storage.local` to store the global pinning mode preference.

* **Background Scripts (`background.js`)**:
    * Handles listening for and dispatching Chrome commands (keyboard shortcuts).
    * When relaying shortcut commands to `content.js`, it passes the appropriate action type (e.g., navigating to a bookmark in a specific mode).

* **Styles (`styles.css`)**:
    Provides styling for the pin option on text selection, the pins themselves on the page, and the Popup interface.

* **Icons**:
    Provides PNG icons in 16x16, 32x32 (optional), 48x48, and 128x128 sizes, typically **placed within an `images` folder in the project and correctly referenced by path in `manifest.json`** to ensure proper loading across various environments.

## How to Use

1.  Download the ChatPin zip and manually load the unpacked extension.
2.  Open a webpage, for example, an LLM conversation page.
3.  Select a piece of text you want to mark.
4.  Click the small “📌” icon that appears to the right of your selection to create a bookmark.
5.  Click the ChatPin icon in your browser toolbar, or use the shortcut `Command+M` / `Ctrl+M`, to open the popup window.
    * You will see a list of bookmarks created on the current page.
    * You can select "Multi-pin" or "Single-pin" mode.
6.  Click a bookmark item in the Popup to automatically scroll to its corresponding position on the page.
7.  Use keyboard shortcuts to quickly navigate to bookmarks.
8.  Directly click a pin icon on the page to delete that bookmark.

## Future Possible Improvements

* Support for persistent bookmarks across pages/sessions (as an optional mode).
* Bookmark export/import functionality.
* More customization options for preview text.
* Ability to add notes/tags to pins.
* Cloud synchronization features.

We hope ChatPin becomes a helpful assistant in your Browse and usage of LLM services!
---
by the way，i never learned code thing,this small extenstion is bulid with cursor ,very interesting to make my idea be true and.


