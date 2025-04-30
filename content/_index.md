---
toc: false
---

<div style="
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    min-height: 80vh;
    padding: 20px;
    max-width: 1200px;
    margin: 0 auto;
">
    <h1 style="margin-bottom: 2rem;">Welcome to my website!</h1>
    <h2 style="margin-bottom: 2rem;">Explore</h2>
    <!-- 四宫格卡片容器 -->
    <div style="
        text-decoration: none !important;
        display: grid;
        grid-template-columns: repeat(2, 1fr); /* 2列 */
        grid-template-rows: repeat(2, 1fr);    /* 2行 */
        gap: 20px;                             /* 卡片间距 */
        width: 100%;
        max-width: 600px;                      /* 限制最大宽度，避免太宽 */
    ">
        {{< card link="blog" title="Blog" icon="book-open" >}}
        {{< card link="about" title="About" icon="user" >}}
    </div>
</div>