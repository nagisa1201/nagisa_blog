---
title: 关于我
categories: 
  - About Me
tags: [我]
date: 2025-09-05
---
<div align="center" style="font-size: 36px; font-weight: 800;">
  你会发现一个有趣的我
</div>

<!-- 1. 头部动态内容：用div包裹隔离主题样式，路径适配网页根目录 -->
<div align="center" style="margin: 0 auto; max-width: 1200px; padding: 0 15px;">
  <!-- 动态打字效果：加强样式优先级 -->
  <img 
    src="https://readme-typing-svg.demolab.com?font=Fira+Code&pause=1000&width=435&lines=console.log(%22Hello%2C%20World%22);小田同学欢迎你的到来!;相信我，你并不孤独;即使迷路，也要前进！&center=true&size=27" 
    alt="打字效果" 
    style="display: block; margin: 0 auto 25px auto !important; max-width: 100%;" 
  />

  <!-- 敲代码图片：网页路径无需../，直接用根目录/ -->
  <picture style="display: block; max-width: 100%; margin: 0 auto;">
    <source media="(prefers-color-scheme: dark)" srcset="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/coding.gif" />
    <source media="(prefers-color-scheme: light)" srcset="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/developer.svg" />
    <img 
      src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/coding.gif" 
      alt="敲代码示意图" 
      style="max-width: 100%; height: auto; max-height: 225px !important; margin: 0 auto;" 
    />
  </picture>

<!-- 个人链接徽章：左右结构 + 最小间距 -->
<div style="
  margin-top: 20px; 
  display: flex; 
  align-items: center; /* 垂直居中 */
  gap: 20px; /* 终极最小间距（可改为1px，再小可能重叠） */
  /* 可选：若想让整体居左，删除下面2行；若想整体居中，保留 */
  width: fit-content; /* 让容器宽度“贴合徽章”，不被页面拉宽 */
  margin-left: auto; margin-right: auto; /* 整体居中（可选） */
">
  <!-- 左侧：博客链接（固定在左） -->
  <a href="https://tlf-nagisa-blog.com/" style="text-decoration: none !important;">
    <img src="https://img.shields.io/badge/Website-博客-8c36db" alt="个人博客" style="vertical-align: middle;" />
  </a>
  <!-- 右侧：B站链接（强制贴紧左侧，无多余空隙） -->
  <a href="https://space.bilibili.com/1792208251" style="text-decoration: none !important; margin-left: auto;">
    <img src="https://img.shields.io/badge/Bilibili-B站-ff69b4" alt="B站账号" style="vertical-align: middle;" />
  </a>
</div>

  <!-- 贪吃蛇贡献图：控制宽度适配网页 -->
  <div style="margin-top: 25px; width: 100%; max-width: 1500px; margin: 25px auto 0 auto;">
    <picture style="display: block; max-width: 100%;">
      <source media="(prefers-color-scheme: dark)" srcset="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/profile-snake-contrib/github-contribution-grid-snake-dark.svg" />
      <source media="(prefers-color-scheme: light)" srcset="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/profile-snake-contrib/github-contribution-grid-snake.svg" />
      <img 
        src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/profile-snake-contrib/github-contribution-grid-snake-dark.svg" 
        alt="GitHub 贪吃蛇贡献图" 
        style="width: 100%; height: auto !important;" 
      />
    </picture>
  </div>
</div>

<!-- 2. 个人介绍与博客：核心修复本地图片路径（../→/）+ 隔离主题样式 -->
## 🙋 Hello
<div style="max-width: 1200px; margin: 0 auto; padding: 0 15px;">

### 🤺 About Me
<!-- 关键：博客网页用根目录路径 /blog-img/，不是本地的 ../blog-img/ -->
<!-- 用overflow:hidden包裹，避免主题浮动样式冲突 -->
<div style="overflow: hidden; margin-bottom: 20px;">
  <img src="/blog-img/introduction/me.jpg" 
       alt="我的照片" 
       style="float: right; width: 150px; height: auto; margin-left: 15px; margin-bottom: 10px; border: none !important;" 
  />

  <ul style="list-style-type: disc; margin-left: 20px; padding-left: 0 !important;">
    <li>嗨，你好，我是小田同学，目前就读于南京理工大学自动化学院，大三。热爱编程、创新、学习、读书、运动、旅行、音乐。</li>
    <li>热爱机器人和计算机，希望能成为一名优秀的开发者。</li>
    <li>日常里，我也是个泡工位敲代码的“技术一般”宅；但，我的其余生活同样丰富：
      <ul style="list-style-type: circle; margin-left: 40px;">
        <li>坚持数年如一日的夜跑；</li>
        <li>闲暇时间看番、剧本杀、探店、摄影；</li>
        <li>还有时常在寝室吃灰的电吉他：(</li>
      </ul>
    </li>
    <li>和同学们去KTV是我的热爱，也会在下课后的饭点儿和好哥们一起吃饭。</li>
    <li>当然啦，有着和大多数同学一样赚钱与经济独立的愿望；与此同时，也希望能够靠着自己过去、当下、未来所学，能对世界有所影响 ：)</li>
  </ul>
</div>


### 📃 Recent Blog
<!-- 同样用overflow:hidden包裹，路径改根目录/ -->
<div style="overflow: hidden; margin-bottom: 20px;">
  <img src="/blog-img/introduction/mygo1.png" 
       alt="博客封面" 
       style="float: right; width: 80px; height: auto; margin-left: 15px; margin-bottom: 10px; border: none !important;" 
  />

  <ul style="list-style-type: disc; margin-left: 20px; padding-left: 0 !important;">
    <li>Sep 1 - <a href="https://tlf-nagisa-blog.com/2025/08/31/tf/" style="color: #0066cc !important; text-decoration: underline !important;">机器人定位基础（算法组第八课）</a></li>
    <li>Mar 19 - <a href="https://tlf-nagisa-blog.com/2025/03/18/slam1/" style="color: #0066cc !important; text-decoration: underline !important;">SLAM部署与imu标定</a></li>
    <li>Apr 24 - <a href="https://tlf-nagisa-blog.com/2025/04/23/idea1/" style="color: #0066cc !important; text-decoration: underline !important;">小牢骚</a></li>
  </ul>
</div>
</div>


<!-- 3. 分割线与中间图：路径改根目录/，适配网页宽度 -->
<!-- ########################################## 分割 ########################################## -->
<img src="/blog-img/introduction/gif1.gif" width="100%" alt="分割动图" style="max-width: 1200px; margin: 0 auto; display: block;" />

<div align="center" style="max-width: 1200px; margin: 0 auto; padding: 0 15px;">
  <img src="/blog-img/introduction/github.png" alt="Metrics" width="80%" max-width="400px" style="margin: 20px auto !important;" />
</div>

<!-- ########################################## 分割 ########################################## -->
<img src="/blog-img/introduction/gif1.gif" width="100%" alt="分割动图" style="max-width: 1200px; margin: 0 auto; display: block;" />


<!-- 4. 名人名言：加强居中样式 -->
<div align="center" style="max-width: 1200px; margin: 20px auto; padding: 0 15px;">
  <img src="https://quotes-github-readme.vercel.app/api?type=horizontal&theme=dark" alt="名人名言" width="80%" style="max-width: 800px; margin: 0 auto;" />
  <p style="color: #999; font-size: 14px; margin-top: 10px !important;">若图片未显示，可刷新页面或检查网络</p>
</div>

<!-- ########################################## 分割 ########################################## -->
<img src="/blog-img/introduction/gif1.gif" width="100%" alt="分割动图" style="max-width: 1200px; margin: 0 auto; display: block;" />


<!-- 5. 技能栈与工具：用flex替代inline-block，避免主题样式冲突 -->
### 🛠️ 技能栈
<div style="max-width: 1200px; margin: 0 auto; padding: 0 15px;">
  <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 8px; margin: 15px 0 !important;">
    <img src="https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=fff&style=flat" alt="HTML5" title="HTML5" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=fff&style=flat" alt="CSS3" title="CSS3" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=000&style=flat" alt="JavaScript" title="JavaScript" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=fff&style=flat" alt="Python" title="Python" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Spring-6DB33F?logo=spring&logoColor=fff&style=flat" alt="Spring" title="Spring" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Qt-41CD52?logo=qt&logoColor=fff&style=flat" alt="Qt" title="Qt" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/C-A8B9CC?logo=c&logoColor=fff&style=flat" alt="C" title="C" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/C%2B%2B-00599C?logo=cplusplus&logoColor=fff&style=flat" alt="C++" title="C++" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/C%20Sharp-239120?logo=csharp&logoColor=fff&style=flat" alt="C#" title="C#" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=000&style=flat" alt="Linux" title="Linux" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Windows-0078D6?logo=windows&logoColor=fff&style=flat" alt="Windows" title="Windows" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Visual%20Studio%20Code-007ACC?logo=visualstudiocode&logoColor=fff&style=flat" alt="VS Code" title="VS Code" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Adobe%20Photoshop-31A8FF?logo=adobephotoshop&logoColor=fff&style=flat" alt="PS" title="Adobe Photoshop" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/Visual%20Studio-5C2D91?logo=visualstudio&logoColor=fff&style=flat" alt="VS" title="Visual Studio" style="vertical-align: middle;" />
    <img src="https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=fff&style=flat" alt="GitHub" title="GitHub" style="vertical-align: middle;" />
  </div>
</div>

### 🧰 常用工具
<div style="max-width: 1200px; margin: 0 auto; padding: 0 15px;">
  <!-- 用flex布局，避免inline-block受主题空格影响 -->
  <div style="display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin: 15px 0 !important;">
    <!-- 第一组 -->
    <div style="display: flex; align-items: center; gap: 5px;">
      <img src="https://skillicons.dev/icons?i=ps,ai,pr,c,cpp,cs,ts,discord,twitter,mongodb,instagram,idea,git" alt="工具图标1" style="height: 65px; width: auto;" />
    </div>
    <!-- 第二组 -->
    <div style="display: flex; align-items: center; gap: 5px;">
      <img src="https://techstack-generator.vercel.app/kubernetes-icon.svg" alt="K8s" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/js-icon.svg" alt="JS" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/mysql-icon.svg" alt="MySQL" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/webpack-icon.svg" alt="Webpack" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/docker-icon.svg" alt="Docker" width="65" style="height: 65px; object-fit: contain;" />
    </div>
    <!-- 第三组 -->
    <div style="display: flex; align-items: center; gap: 5px;">
      <img src="https://techstack-generator.vercel.app/redux-icon.svg" alt="Redux" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/java-icon.svg" alt="Java" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/eslint-icon.svg" alt="ESLint" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/aws-icon.svg" alt="AWS" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/ts-icon.svg" alt="TS" width="65" style="height: 65px; object-fit: contain;" />
      <img src="https://techstack-generator.vercel.app/nginx-icon.svg" alt="Nginx" width="65" style="height: 65px; object-fit: contain;" />
    </div>
    <!-- 动图组 -->
    <div style="display: flex; align-items: center; gap: 5px; margin-top: 10px;">
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/html.webp" alt="HTML" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/cssgif.webp" alt="CSS" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/vscode.webp" alt="VS Code" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/react.webp" alt="React" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/vue.webp" alt="Vue" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/python.webp" alt="Python" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/js.webp" alt="JS" width="100" height="100" style="object-fit: contain;" />
      <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/github.webp" alt="GitHub" width="100" height="100" style="object-fit: contain;" />
    </div>
  </div>
</div>


<!-- ########################################## 分割 ########################################## -->
<img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/hr.gif" width="100%" alt="底部分割线" style="max-width: 1200px; margin: 0 auto; display: block;" />


<!-- 6. 底部图标：适配网页居中 -->
<div align="center" style="max-width: 1200px; margin: 20px auto; padding: 0 15px;">
  <img src="https://cdn.jsdelivr.net/gh/sun0225SUN/sun0225SUN/assets/images/icon.png" alt="GitHub 指标图标" style="max-width: 700px; margin: 0 auto;" />
  <p style="color: #999; font-size: 14px; margin-top: 10px !important;">感谢阅读，期待与你交流！</p>
</div>