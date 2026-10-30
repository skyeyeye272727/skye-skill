# SKYE skill

SKYE 是公司電腦上的執行層：讓 ChatGPT（或其他 AI）操作這台 Windows 電腦的檔案、App、瀏覽器、文字檔與 PDF。

**只裝這個 skill 不會動。** 必須先在電腦上安裝 SKYE 程式，skill 才有東西可以叫。

## 同事安裝（Windows）
1. 檔案總管網址列貼上：`\\192.168.3.3\【公共檔案】\〔行銷部〕\【資訊組】\SKYE 安裝包`
2. 把「SKYE 安裝包 x64.zip」複製到桌面、解壓縮，雙擊「安裝 SKYE.cmd」。
3. 安裝程式會自動把這個 skill 和 MCP 設定寫進 ChatGPT 桌面版。重開 ChatGPT，在 Plugins → Skills 看得到「SKYE」就完成。

## 手動安裝 skill（已裝 SKYE 程式、但 skill 不見時）
把 `skills/skye` 整個資料夾複製到：
- Windows：`%USERPROFILE%\.agents\skills\skye`
- macOS：`~/.agents/skills/skye`

需要能讀本機 skill 的 ChatGPT 桌面版（網頁版讀不到）。

Copyright © 2026 Skye. All rights reserved.
