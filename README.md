# about
一個簡單的 epub 範本，供修改與練習用。
> 內容說明請參考 [wiki](https://github.com/air790725/epub_example/wiki) 頁面

## 發佈 epub
發佈 epub 時，整個專案目錄會封裝成一個 .zip 檔，我們可以透過指令封裝 epub
```
zip -0 -X xxx.epub mimetype
zip -9 -r xxx.epub */
```
參數說明：
```
-0~9  // 壓縮級別：數字越大，壓縮率越高
-X    // 不保存額外的文件屬性
-r    // 將指定目錄下的所有文件和子目錄一併處理
```
