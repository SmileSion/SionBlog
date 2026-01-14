{{/* --- 🎨 30 种高级色谱 (文字情绪分类检索版) --- */}}
{{- $accent := "#7B8B9C" -}} {{/* 默认：远山蓝 */}}

{{/* A. 理性、思考、淡淡的忧郁 (适合：随笔、深度思考、哲学独白) */}}
{{- if eq $mood "fog" -}}         {{$accent = "#7B8B9C"}}{{/* 远山蓝：冷静客观的思考 */}}
{{- else if eq $mood "mist" -}}    {{$accent = "#9DA5A8"}}{{/* 迷雾灰：迷茫或模糊的记忆 */}}
{{- else if eq $mood "plum" -}}    {{$accent = "#9079AD"}}{{/* 烟梅色 dreamy：感性与理性的边界 */}}
{{- else if eq $mood "indigo" -}}  {{$accent = "#5D7599"}}{{/* 靛青色：深夜的静思 */}}
{{- else if eq $mood "sand" -}}    {{$accent = "#BDB3A7"}}{{/* 浅沙色：极简、空无、留白 */}}

{{/* B. 感性、悸动、柔软的怀念 (适合：情书、温柔时刻、怀旧往事) */}}
{{- else if eq $mood "sakura" -}}  {{$accent = "#F4B5AD"}}{{/* 初樱粉：初见或美好的瞬间 */}}
{{- else if eq $mood "rose" -}}    {{$accent = "#B47E88"}}{{/* 枯玫瑰：破碎但美丽的怀念 */}}
{{- else if eq $mood "camellia" -}}{{$accent = "#D68D9A"}}{{/* 山茶色：热烈而含蓄的爱意 */}}
{{- else if eq $mood "petal" -}}   {{$accent = "#E8C3C9"}}{{/* 浅瓣粉：少女心、微甜的日常 */}}
{{- else if eq $mood "dusk" -}}    {{$accent = "#C1969B"}}{{/* 晚霞粉：落日余晖下的温柔 */}}

{{/* C. 叙事、厚重、古典的质感 (适合：生活史、传统文化、沉稳的纪录) */}}
{{- else if eq $mood "vermilion" -}}{{$accent = "#B35C44"}}{{/* 朱砂红：传统的喜悦或庄重 */}}
{{- else if eq $mood "cyan" -}}     {{$accent = "#5C8A8A"}}{{/* 黛蓝色：如山水画般的宁静 */}}
{{- else if eq $mood "gold" -}}     {{$accent = "#AD9152"}}{{/* 琥珀金：岁月的沉淀与光泽 */}}
{{- else if eq $mood "jade" -}}     {{$accent = "#61856E"}}{{/* 苍翠色：古老、坚韧、自然 */}}
{{- else if eq $mood "ink" -}}      {{$accent = "#4F5355"}}{{/* 徽墨黑：书卷气、深沉的自白 */}}

{{/* D. 平和、踏实、生活的气息 (适合：日常小记、散步记录、平凡生活) */}}
{{- else if eq $mood "wood" -}}    {{$accent = "#8C7B6C"}}{{/* 檀木棕：像木头一样稳固的情绪 */}}
{{- else if eq $mood "clay" -}}    {{$accent = "#A68069"}}{{/* 赭石土：大地般的包容感 */}}
{{- else if eq $mood "moss" -}}    {{$accent = "#7D8259"}}{{/* 苔藓绿：不起眼但顽强的生命 */}}
{{- else if eq $mood "cedar" -}}   {{$accent = "#958172"}}{{/* 雪松木：清晨森林的木质香 */}}
{{- else if eq $mood "autumn" -}}  {{$accent = "#BD8260"}}{{/* 晚秋橘：收获后的从容与知足 */}}

{{/* E. 克制、疏离、冷峻的孤独 (适合：独处、雪天、清冷的清晨) */}}
{{- else if eq $mood "lake" -}}    {{$accent = "#6B8E8E"}}{{/* 静谧湖：深不见底的宁静 */}}
{{- else if eq $mood "ice" -}}     {{$accent = "#A5B9C7"}}{{/* 碎冰蓝：清澈、寒冷、透明 */}}
{{- else if eq $mood "deep" -}}    {{$accent = "#4A677D"}}{{/* 深海蓝：极致的孤独与辽阔 */}}
{{- else if eq $mood "mint" -}}    {{$accent = "#8FAEA3"}}{{/* 寒薄荷：清醒得甚至有些刺骨 */}}
{{- else if eq $mood "winter" -}}  {{$accent = "#B2B8C1"}}{{/* 冬云灰：阴天里的沉默 */}}

{{/* F. 治愈、明亮、向上的能量 (适合：鼓励、快乐瞬间、清新的愿望) */}}
{{- else if eq $mood "matcha" -}}  {{$accent = "#95A472"}}{{/* 抹茶色：呼吸感、放松的身心 */}}
{{- else if eq $mood "sunset" -}}  {{$accent = "#D49A89"}}{{/* 暖落日：被世界温柔包裹 */}}
{{- else if eq $mood "honey" -}}   {{$accent = "#C9A16A"}}{{/* 蜂蜜黄：生活里的甜与光亮 */}}
{{- else if eq $mood "lavender" -}}{{$accent = "#A7A1C1"}}{{/* 薰衣草：浪漫而平静的梦境 */}}
{{- else if eq $mood "lemon" -}}   {{$accent = "#D9C175"}}{{/* 柠檬盐：清爽、活泼、有朝气 */}}
{{- end -}}

### 如果你想让我帮你生成（提示词）

如果你以后想把一段话发给我，让我按照这个风格优化排版，你可以直接复制并使用下面的**“提示词指令”**：

> **提示词：**
> “请按照‘苹果杂志编辑风格’重排下面这段碎碎念。要求：
> 1. 提取一个富有情绪张力的短标题。
> 2. 将全文拆分为：导语、正文、独白大字、结尾四个层次。
> 3. 语言风格保持克制、平实，但保留文字原有的破碎感或力量感。
> 4. 直接输出 Markdown，使用我预设的 `{{< thought >}}` shortcode 包装。
> 
> 
> **文字内容：** [此处粘贴你的文字]”
