---
# the default layout is 'page'
icon: fas fa-link
order: 4
---
<div class="friend-links-container">
  {% for friend in site.data.friends %}
    <a href="{{ friend.url }}" target="_blank" rel="noopener" class="friend-card">
      <div class="friend-card-avatar">
        <img src="{{ friend.avatar }}" alt="{{ friend.name }}'s avatar" loading="lazy" onerror="this.src='https://www.google.com/s2/favicons?domain={{ friend.url | split: '//' | last | split: '/' | first }}'; this.onerror=null;" />
      </div>
      <div class="friend-card-info">
        <strong class="friend-card-name">{{ friend.name }}</strong>
        <p class="friend-card-desc">{{ friend.desc }}</p>
      </div>
    </a>
  {% endfor %}
</div>

<style>
  /* 确保所有元素盒模型统一，避免 padding 导致尺寸混乱 */
  .friend-links-container *,
  .friend-links-container *::before,
  .friend-links-container *::after {
    box-sizing: border-box;
  }

  /* 友链网格布局容器 */
  .friend-links-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 1rem; /* 卡片间距 */
  }

  /* 核心：卡片本身 (<a> 标签) */
  .friend-card {
    display: flex; /* <-- 这是让头像和信息并排的关键！*/
    align-items: center; /* 垂直居中对齐 */
    text-decoration: none !important; /* !important 用来强制去掉下划线 */
    color: #333; /* 默认文字颜色 */
    background-color: #fff; /* 卡片背景色 */
    padding: 1rem;
    border-radius: 12px;
    border: 1px solid #eee; /* 添加一个细边框，看起来更精致 */
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.03);
    transition: transform 0.3s ease, box-shadow 0.3s ease, border-color 0.3s ease;
  }

  /* 鼠标悬浮效果 */
  .friend-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
    border-color: #ddd;
  }

  /* 卡片头像容器 */
  .friend-card-avatar {
    flex-shrink: 0; /* 防止头像被压缩 */
    width: 55px;
    height: 55px;
    margin-right: 1rem; /* 头像和文字之间的距离 */
  }

  /* 头像图片 */
  .friend-card-avatar img {
    width: 100%;
    height: 100%;
    border-radius: 50%; /* 圆形头像 */
    object-fit: cover; /* 保证图片不变形 */
    border: 1px solid #f0f0f0; /* 给头像加个细边框 */
  }

  /* 卡片信息区域 */
  .friend-card-info {
    overflow: hidden; /* 防止文字溢出 */
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  /* 网站名称 */
  .friend-card-name {
    font-size: 1.1em;
    font-weight: bold;
    margin: 0;
    padding: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  /* 网站描述 */
  .friend-card-desc {
    font-size: 0.9em;
    color: #888;
    margin: 5px 0 0 0; /* 与标题的上边距 */
    padding: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>
