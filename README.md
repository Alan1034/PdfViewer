# GeneralPdfViewer

一个前端展示PDF的组件

![image-20241107094814662](https://raw.githubusercontent.com/Alan1034/PicturesServer/main/PicGo_imgs/202411070951481.png)

```
import { VPdfViewer } from "general-pdf-viewer"
import 'general-pdf-viewer/style'
```

```
<VPdfViewer :visable.sync="pdfVisable"
componentType="Tdesign Mobile Vue"
:path="pdfPath"></VPdfViewer>
```

visable：控制是否展示

path：文件路径

componentType: UI组件，支持Element Plus（默认）、Tdesign Mobile Vue