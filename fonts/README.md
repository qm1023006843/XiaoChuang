# fonts

小窗「外观 → 字体 → 文泉驿正黑 + Inter」用的字体文件。跟页面放在同一个地方，不找第三方要；
选「系统字体」时一个字节都不会下载。

| 文件 | 来源 | 内容 | 授权 |
| --- | --- | --- | --- |
| `inter-400/500/600/700.woff2` | [Inter](https://github.com/rsms/inter) 4.0 | 英文、数字、西文标点（含 “ ” ‘ ’ … —） | SIL OFL 1.1，见 `OFL.txt` |
| `wqy-zenhei-gb2312.woff2` | [文泉驿正黑](http://wenq.org/) 0.9.45 | GB2312 的 6763 个常用汉字和全角标点 | GPL v2 + 字体嵌入例外，见下方和 `GPL-2.txt` |

两个都是从原字体裁出来的子集（只留上面那些字，去掉点阵和 hinting），没有改字形。
GB2312 以外的生僻字会退回系统字体（iPhone 上是苹方）。

## 怎么重新生成

原字体取自 Debian 的 `fonts-inter`、`fonts-wqy-zenhei` 包，工具是 [fonttools](https://github.com/fonttools/fonttools)
（`pip install fonttools brotli`）。

```sh
# 文泉驿正黑：GB2312 全部字符，去掉 Inter 已经覆盖的 ASCII / Latin-1 / 通用标点
python3 - <<'EOF'
cps = set()
for hi in range(0xA1, 0xF8):
    for lo in range(0xA1, 0xFF):
        try: cps.add(ord(bytes([hi, lo]).decode("gb2312")))
        except UnicodeDecodeError: pass
cps -= set(range(0x20, 0x7F)) | set(range(0xA0, 0x100)) | set(range(0x2000, 0x2070))
cps |= {0x3000, 0x30FB, 0xFF5E}
open("gb2312.txt", "w").write("\n".join("U+%04X" % c for c in sorted(cps)))
EOF
pyftsubset /usr/share/fonts/truetype/wqy/wqy-zenhei.ttc --font-number=0 --unicodes-file=gb2312.txt \
  --drop-tables+=EBDT,EBLC,EBSC --no-hinting --desubroutinize --name-IDs='*' --name-languages='*' \
  --flavor=woff2 --output-file=wqy-zenhei-gb2312.woff2

# Inter：四个字重各一份
for w in Regular:400 Medium:500 SemiBold:600 Bold:700; do
  pyftsubset /usr/share/fonts/opentype/inter/Inter-${w%%:*}.otf \
    --unicodes="U+0020-007E,U+00A0-00FF,U+0131,U+0152-0153,U+02C6,U+02DA,U+02DC,U+2000-206F,U+20AC,U+2122,U+2212" \
    --layout-features='kern,liga,calt,ccmp,locl,mark,mkmk' --no-hinting --name-IDs='*' --name-languages='*' \
    --flavor=woff2 --output-file=inter-${w##*:}.woff2
done
```

## 文泉驿正黑的授权原文

Copyright: 2007-2009, Qianqian Fang and The WenQuanYi Project Board of Trustees;
1998-2004, Koanughi Un; 1999-2004, Tsong-Min Wu, Tsong-Huey Wu;
2005, Tsong-Min Wu, Tsong-Huey Wu and Edward G.J. Lee; 2004, Firefly; 2002-2008, M+ FONTS PROJECT

> This package is free software; you can redistribute it and/or modify it under the terms of the
> GNU General Public License as published by the Free Software Foundation; either version 2 of the License.
>
> This package is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even
> the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public
> License for more details.
>
> As a special exception, if you create a document which uses this font, and embed this font or unaltered
> portions of this font into the document, this font does not by itself cause the resulting document to be
> covered by the GNU General Public License. This exception does not however invalidate any other reasons
> why the document might be covered by the GNU General Public License. If you modify this font, you may
> extend this exception to your version of the font, but you are not obligated to do so. If you do not wish
> to do so, delete this exception statement from your version.

其中来自 M+ FONTS 的字形另适用 M+ FONTS License：

> These fonts are free software. Unlimited permission is granted to use, copy, and distribute it, with or
> without modification, either commercially and noncommercially. THESE FONTS ARE PROVIDED "AS IS" WITHOUT WARRANTY.

裁剪后的子集沿用同样的授权（含字体嵌入例外）。
