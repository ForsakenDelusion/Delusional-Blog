---
# the default layout is 'page'
icon: fas fa-link
order: 4
---

<div class="friend-links-container">
  {% for friend in site.data.friends %}
    <a href="{{ friend.url }}" target="_blank" class="friend-card" title="{{ friend.name }}">
      <div class="friend-card-avatar">
        <img src="{{ friend.avatar }}" alt="{{ friend.name }}'s avatar" onerror="this.src='https://www.google.com/s2/favicons?domain={{ friend.url | split: '//' | last | split: '/' | first }}'; this.onerror=null;" />
      </div>
      <div class="friend-card-info">
        <div class="friend-card-name">{{ friend.name }}</div>
        <div class="friend-card-desc">{{ friend.desc }}</div>
      </div>
    </a>
  {% endfor %}
</div>

<style>
  .friend-links-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 20px; /* 卡片之间的间距 */
  }

  .friend-card {
    display: flex;
    align-items: center;
    padding: 15px;
    border-radius: 8px;
    background-color: #f8f9fa; /* 卡片背景色，你可以根据你的主题调整 */
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    transition: all 0.3s ease;
    color: inherit; /* 继承父元素的文字颜色 */
    text-decoration: none; /* 去掉下划线 */
  }

  .friend-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    background-color: #ffffff; /* 鼠标悬浮时的背景色 */
  }

  .friend-card-avatar {
    flex-shrink: 0;
    width: 50px;
    height: 50px;
    margin-right: 15px;
  }

  .friend-card-avatar img {
    width: 100%;
    height: 100%;
    border-radius: 50%; /* 圆形头像 */
    object-fit: cover;
    background-color: #fff; /* 防止透明背景的png图片看起来奇怪 */
  }

  .friend-card-info {
    overflow: hidden; /* 防止文字过长溢出 */
  }

  .friend-card-name {
    font-size: 1.1em;
    font-weight: bold;
    color: #333;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis; /* 名字太长时显示省略号 */
  }

  .friend-card-desc {
    font-size: 0.9em;
    color: #6c757d;
    margin-top: 5px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis; /* 简介太长时显示省略号 */
  }
</style>
