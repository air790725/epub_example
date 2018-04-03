# about epub
一個基本的 epub 須包含幾個特定的目錄及檔案，檔名與檔案位置皆受到嚴格規範：
```
META-INF/
  container.xml
OEBPS/
  content.opf
  toc.ncx
mimetype
```
> OEBPS (Open eBook Publication Structure) 目錄並非必要規範，但一般建議這麼做，將與電子書內容相關的檔案 (如 xhtml, css, images...) 放入 `OEBPS/` 中。

## The mimetype file
宣告電子書檔案類型，內容僅一行 `application/epub+zip` ，不能含有其他空白行

## META-INF/container.xml
打開電子書時閱讀軟體優先讀取的檔案，由這隻檔案指向電子書的 metadata 位置。`container.xml` 檔案內容如下：
```xml
<?xml version="1.0"?>
<container version="1.0" xmlns="urn:oasis:names:tc:opendocument:xmlns:container">
  <rootfiles>
    <rootfile full-path="OEBPS/content.opf" media-type="application/oebps-package+xml" />
  </rootfiles>
</container>
```
`full-path` 是唯一可能變動的值，視 `content.opf` 位置而異，通常建議放在 `OEBPS` 目錄下。要注意的是：路徑是相對於 epub 專案頂層，而非相對於 META-INF 目錄。除了 container.xml 以外，META-INF 也能包含其他檔案，允許 epub 支援更多項目，例如：
```
manifest.xml    // 文件列表
metadata.xml    // 後設資料
signatures.xml  // 數位簽章
encryption.xml  // 加密 
rights.xml      // 數位版權管理 (DRM)
```

## opf (Open Packaging Format) metadata file
opf 為 epub 必要檔案，檔名雖可更改，但普遍命名為 `content.opf`，負責指出電子書內容的檔案位置，如 xhtml 檔、image 檔等，也指向其他 metadata 檔案 (Navigation Center eXtended, NCX)。內容大致如下：
```xml
<?xml version='1.0' encoding='utf-8'?>
<package xmlns="http://www.idpf.org/2007/opf"
            xmlns:dc="http://purl.org/dc/elements/1.1/"
            unique-identifier="bookid" version="2.0">
  <metadata>
    <dc:title>BOOK TITLE</dc:title>
    <dc:creator>BOOK AUTHOR</dc:creator>
    <dc:identifier id="bookid">urn:uuid:123-456-789</dc:identifier>
    <dc:language>en-US</dc:language>
    <meta name="cover" content="cover-image" />
  </metadata>
  <manifest>
    <item id="cover" href="cover.xhtml" media-type="application/xhtml+xml"/>
    <item id="content" href="content.xhtml" media-type="application/xhtml+xml"/>
    <item id="cover-image" href="images/cover.png" media-type="image/png"/>
    <item id="css" href="stylesheet.css" media-type="text/css"/>
    <item id="ncx" href="toc.ncx" media-type="application/x-dtbncx+xml"/>
  </manifest>
  <spine toc="ncx">
    <itemref idref="cover" linear="no"/>
    <itemref idref="content"/>
  </spine>
  <guide>
    <reference href="cover.xhtml" type="cover" title="Cover"/>
  </guide>
</package>
```

### opf schemas & namespaces
opf 檔案須使用 `http://www.idpf.org/2007/opf` 作為命名空間，且其 metadata 須在 DCIM (Dublin Core Metadata Initiative) 的命名空間 `http://purl.org/dc/elements/1.1/` 底下

### metadata
Dublin Core 定義一組常用的項目來描述多種數位內容 (不屬於 epub 規範)，這些項目可在 opf 中的 `<metadata>` 標籤中被定義：
```xml
...
<metadata>
  <dc:title>BOOK TITLE</dc:title>
  <dc:creator>BOOK AUTHOR</dc:creator>
  <dc:identifier id="bookid">urn:uuid:123-456-789</dc:identifier>
  <meta name="cover" content="cover-image" />
</metadata>
...
```
`title` 與 `identifier` 是必要項目。依 epub 規範，identifier 必須是唯一值，由書籍創造者提供。對出版商來說，一般會使用包含 ISBN 在內的一組字串；而對於其他書籍創造者來說，可考慮使用 URL 或一組隨機產生的 unique user ID (UUID)。要注意的是 `dc:identifier` 的 id 屬性須對應 package 的 `unique-identifier` 屬性。

其他的 metadata 項目如下，可選擇是否增加：
* dc:language
* dc:date
* dc:publisher
* dc:rights (If releasing the work under a Creative Commons license, put the URL for the license here.)

> 加入一個 `name = ”cover”` 的 meta 元素並非 epub 直接規範，但一般建議用這種方式使封面頁較容易被找到。有些 epub 閱讀器喜歡使用圖片為封面頁，但也有部分使用內含圖片的 xhtml。上面的範例直接指向圖片，meta 元素的 content 屬性須和 `manifest` 裡的圖片檔 id 屬性一致。


### manifest
opf 的 manifest 列出所有在 epub 內容中可被找到的資源(含 metadata)，通常指的就是列出電子書中包含文本內容的 xhtml 加上相關的媒體檔案(例如圖片)。此外 epub 鼓勵透過 css 來編排書籍內容，因此 css 檔案也必須被包含進 manifest 裡。也就是說，所有會進到電子書的檔案都應被列在 manifest 中。
```xml
...
<manifest>
  <item id="ncx" href="toc.ncx" media-type="application/x-dtbncx+xml"/>
  <item id="cover" href="cover.xhtml" media-type="application/xhtml+xml"/>
  <item id="content" href="content.xhtml" media-type="application/xhtml+xml"/>
  <item id="cover-image" href="images/cover.png" media-type="image/png"/>
  <item id="css" href="stylesheet.css" media-type="text/css"/>
</manifest>
...
```

> 進階的 manifest 範例可能包含更多的 xhtml、css 、image 等，無論如何，最後都要記得加入 `toc.ncx`。另外要注意的是：所有項目都有個適當的 media-type，這個值是必要的，且 xhtml 的 media-type 必須為 `application/xhtml+xml`。

epub 支援四種核心的影像格式：
* jpeg
* png
* gif
* svg

### spine
manifest 已經告訴閱讀器哪些檔案是電子書的內容，接著再透過 spine 告訴閱讀器這些內容的讀取順序(即閱讀順序)
```xml
...
<spine toc="ncx">
  <itemref idref="cover" linear="no"/>
  <itemref idref="content"/>
</spine>
...
```
* 每個 `itemref` 元素都有一個必要的 `idref` 屬性，必須與 manifest 中 item 的 id 屬性一致。
* spine 的 `toc` 屬性是必須的，它參考 manifest 中指向 ncx 檔案的 item 的 id 屬性。
* linear 屬性指出該 item 是否被視為閱讀順序的一部分。建議封面頁定義為 `linear = “no”`，這樣閱讀時，便不需從封面頁開始翻頁。

### guide
opf 的最後一部分是 guide，雖然非必要，但建議加入。
```xml
...
<guide>
  <reference href="cover.xhtml" type="cover" title="Cover"/>
</guide>
...
```
guide 是一個提供語意資訊給 epub 閱讀系統的管道，底下是一些允許在 opf guide 中列出的部分項目：
* cover: The book cover
* title-page: A page with author and publisher information
* toc: The table of contents

## NCX table of contents
僅管 opf 文件已經定義了一部分，但 epub 還借用了 DAISY 的 NCX DTD。NCX 定義了書籍內容的目錄表格，在複雜的書籍目錄中，它通常是分層的章節和區域，類似巢狀結構。
> DAISY 是個財團，為那些通常因視覺障礙或無法操作印刷品而無法閱讀傳統書籍的讀者發展資料格式
```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE ncx PUBLIC "-//NISO//DTD ncx 2005-1//EN" "http://www.daisy.org/z3986/2005/ncx-2005-1.dtd">
<ncx xmlns="http://www.daisy.org/z3986/2005/ncx/" version="2005-1">
  <head>
    <meta name="dtb:uid" content="urn:uuid:123-456-789"/>
    <meta name="dtb:depth" content="1"/>
    <meta name="dtb:totalPageCount" content="0"/>
    <meta name="dtb:maxPageNumber" content="0"/>
  </head>
  <docTitle>
    <text>BOOK TITLE</text>
  </docTitle>
  <navMap>
    <navPoint id="navpoint-1" playOrder="1">
      <navLabel>
        <text>Book cover</text>
      </navLabel>
      <content src="cover.xhtml"/>
    </navPoint>
    <navPoint id="navpoint-2" playOrder="2">
      <navLabel>
        <text>Contents</text>
      </navLabel>
      <content src="content.xhtml"/>
    </navPoint>
  </navMap>
</ncx>
```

### NCX metadata
DTD 需要在 NCX `<head>` 標籤裡包含 4 個 meta 元素 
* uid: 電子書的唯一ID，應與 opf 檔案中的 dc:identifier 一致
* depth: 反應目錄表格中的分層層級 (level)
* totalPageCount & maxPageNumber: 只應用於紙本書，因此可設為 0 

### docTitle
docTitle 中 `<text>` 的標籤內容應與 opf 檔案中的 `dc:title` 一致
  
### NCX navMap
NCX 與 opf spine 皆描述了電子書文件的內容與順序，較好解釋兩者差異的方式是與紙本書類比：
* opf spine 描述了書的各區塊如何實際地綁在一起，例如將第一章與第二章綁定，那個在第一章的最後一頁即可接續閱讀第二張的第一頁
* NCX 則描述書的目錄表格，如各章節目錄、圖目錄、表目錄⋯⋯等。
> NCX 通常包含更多的 navPoint 元素，超過 opf spine 中有的 itemref 元素；opf spine 中的所有 item 會出現在 NCX 中，但 NCX 可以比 opf spine 更細。

navMap 是 NCX 檔案中最重要的部分，它包含了一個或多個 navPoint 元素。每個 navPoint 元素必包含以下要素：
* playOrder 屬性：文件的閱讀順序
* navLabel/text 元素：文件的標題，一般是章節標題或編號，如第一章⋯⋯等。
* content 元素：`src` 屬性指向文件內容(必須在 opf manifest 中宣告)
* 其他一個或多個 navPoint 子元素：即多層的巢狀結構

## Adding the final content
pub 對於內容並沒有特定的規範，但仍建議採用以下結構：
```
images/     在 OEBPS 目錄底下建立 image 目錄，放置所有圖片檔案
css/        在 OEBPS 目錄底下建立 css 目錄，放置所有 css 檔
js/         在 OEBPS 目錄底下建立 js 目錄，放置所有 js 檔（一般來說用不到）
xxx.xhtml   實際編輯電子書內容的地方
```

### XHTML and CSS in an epub book
底下是一個 epub 內容頁的範例（透過此範例可產生一個封面頁）
```xml
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>BOOK TITLE</title>
    <link type="text/css" rel="stylesheet" href="stylesheet.css" />
  </head>
  <body>
    <h1>BOOK TITLE</h1>
    <div><img src="images/cover.png" alt="cover page"/></div>
  </body>
</html>
```

#### XHTML 內容遵循幾條規則，可能與一般網頁開發不同：
* `img` 元素只能參考專案資料夾的 local 圖檔，無法從外部網站參考
* epub 支援 css 但仍有些微差異(雖然不影響常用的 style)
* epub 未強制要求閱讀器支援 javascript，因此應盡量避免使用 script 區域

## 發佈 epub
發佈 epub 時，整個專案目錄會封裝成一個 .zip 檔，mimetype 必須為 .zip 內的第一個檔案，且本身不可被壓縮。我們可以透過指令封裝 epub
```
zip -0 -X xxx.epub mimetype
zip -9 -r xxx.epub */
```
參數說明：
* -0~9  // 壓縮級別：數字越大，壓縮率越高
* -X    // 不保存額外的文件屬性
* -r    // 將指定目錄下的所有文件和子目錄一併處理
