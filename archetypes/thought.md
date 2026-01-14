+++
title = "{{ replace .Name "-" " " | title }}"
date = {{ .Date }}
draft = true
tags = ["碎碎念"]
+++

{{< posts/thought title="这里是标题" mood="lake" meta="碎碎念 / {{ .Date.Format "01" }}" >}}

这里是导语段落（第一段会自动变大）。例如：我常常反思自己，为什么以前说喜欢我的人，后来渐行渐远。

在此继续书写你的感悟...

{{< img src="image-name.jpg" caption="图片描述" alt="配图说明" >}}

这里是正文部分，**敏感多疑，性格粘人**（这里的加粗在你的主题里会自动带上下划线）。

> “这里放置你想引用的金句或内心独白。”

---

这里是收尾段落。

**这里是最后的强调内容。**

{{< /posts/thought >}}