---
title: JKL27
toc: false
---

<div style="
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    min-height: 80vh;  /* 这会让内容在视口中垂直居中 */
    padding: 20px;
    max-width: 1200px;
    margin: 0 auto;
">
    <h1 style="margin-bottom: 2rem;">Welcome to my website!</h1>
    <h2 style="margin-bottom: 2rem;">Explore</h2>
    <div style="
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 20px;
        width: 100%;
    ">
        {{< cards >}}
            {{< card link="blog" title="Blog" icon="book-open" >}}
            {{< card link="about" title="About" icon="user" >}}
        {{< /cards >}}
    </div>
</div>