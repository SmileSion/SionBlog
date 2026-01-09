+++
date = '2026-01-07T08:00:00+08:00'
draft = false
title = 'About Me'
showtoc = false
tags = ["About","Profile","Welcome"]

+++

欢迎来到SmileSion的博客！

这里记录了我的一些技术文章、摄影作品以及杂七杂八。

希望通过这个博客分享有趣的内容。

---

## 技能 & 爱好

- Go / Python / Web 开发
- Linux / MacOS / 软路由
- 摄影 / 旅行 / 吃 / 二次元 / 游戏

---

## 联系方式

- 邮箱: 1065716478@qq.com  

<a href="/msg-board/" class="floating-contact-globe" id="contactGlobe">
    <div class="globe-content">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-message-circle">
            <path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"></path>
        </svg>
        <span class="globe-text">留言</span>
    </div>
</a>

<style>
/* 悬浮球核心样式 */
.floating-contact-globe {
    position: fixed;
    bottom: 40px;
    right: 30px;
    z-index: 999;
    text-decoration: none !important;
    
    /* 苹果风毛玻璃 */
    background: rgba(255, 255, 255, 0.7);
    -webkit-backdrop-filter: blur(15px);
    backdrop-filter: blur(15px);
    border: 1px solid rgba(255, 255, 255, 0.4);
    
    padding: 12px 20px;
    border-radius: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    
    /* 细腻阴影 */
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08), 0 1px 2px rgba(0, 0, 0, 0.02);
    transition: all 0.4s cubic-bezier(0.23, 1, 0.32, 1);
}

.globe-content {
    display: flex;
    align-items: center;
    gap: 8px;
    color: #1d1d1f; /* 苹果经典文字黑 */
}

.floating-contact-globe svg {
    width: 20px;
    height: 20px;
    transition: transform 0.3s ease;
}

.globe-text {
    font-size: 14px;
    font-weight: 500;
    letter-spacing: 0.5px;
    white-space: nowrap;
}

/* 交互效果 */
.floating-contact-globe:hover {
    background: rgba(255, 255, 255, 0.9);
    transform: translateY(-5px) scale(1.02);
    box-shadow: 0 12px 40px rgba(0, 0, 0, 0.12);
}

.floating-contact-globe:active {
    transform: scale(0.95);
}

/* 滚动时的优雅收缩态 */
.floating-contact-globe.scrolling {
    opacity: 0.5;
    transform: scale(0.9) translateX(10px);
    filter: blur(1px);
}

/* 适配移动端 */
@media (max-width: 768px) {
    .floating-contact-globe {
        bottom: 30px;
        right: 20px;
        padding: 10px 16px;
    }
    .globe-text { font-size: 13px; }
}

/* 深色模式自适应（如果你的博客支持） */
@media (prefers-color-scheme: dark) {
    .floating-contact-globe {
        background: rgba(28, 28, 30, 0.7);
        border-color: rgba(255, 255, 255, 0.1);
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    }
    .globe-content { color: #f5f5f7; }
}
</style>

<script>
    // 监听滚动，实现滚动时收缩的“高级感”
    const globe = document.getElementById('contactGlobe');
    let scrollTimeout;

    window.addEventListener('scroll', function() {
        // 滚动中
        globe.classList.add('scrolling');
        
        // 停止滚动 150ms 后恢复
        clearTimeout(scrollTimeout);
        scrollTimeout = setTimeout(function() {
            globe.classList.remove('scrolling');
        }, 150);
    });
</script>