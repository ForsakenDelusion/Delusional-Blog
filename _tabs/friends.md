---
# the default layout is 'page'
icon: fas fa-link
order: 4
---
欢迎交换友链！请 <a href="mailto:wangcy0205@gmail.com?subject=友链交换请求&body=%0D%0A%0D%0A我的网站信息如下：%0D%0A%0D%0A- 网站名称：%0D%0A- 网站地址：%0D%0A- 头像URL：%0D%0A- 网站描述：%0D%0A%0D%0A">点击这里发送邮件</a> 
<style>
  /* * 网格布局容器
   * 在不同屏幕宽度下自动调整每行的卡片数量
   */
  .page-links {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 20px;
    padding: 10px 0;
    /* --- 优化 1: 合并渐显动画的初始状态 --- */
    visibility: hidden;
    opacity: 0;
    transition: opacity 0.5s ease;
  }

  /* 当容器获得 .loaded 类时，它会变得可见 */
  .page-links.loaded {
    visibility: visible;
    opacity: 1;
  }

  /* * 卡片项容器 (.page-links-item)
   * 负责卡片的背景、阴影、圆角和内部元素的 flex 布局
   * 使用 position: relative 来配合内部的“幽灵”链接
   */
  .page-links-item {
    position: relative;
    display: flex;
    align-items: center;
    padding: 16px;
    border-radius: 12px;
    background-color: var(--card-bg, #fff);
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    gap: 16px;
  }

  .page-links-item:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 24px rgba(0, 0, 0, 0.1);
  }

  /* * 头像相框 (.page-links-avatar-frame)
   * 负责创建头像的边框、形状和悬浮效果
   */
  .page-links-avatar-frame {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 60px;
    height: 60px;
    flex-shrink: 0;
    background-color: var(--card-bg-secondary, #f0f0f0);
    border-radius: 12px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
    transition: transform 0.3s ease;
    overflow: hidden; /* --- 优化 2: 移除调试时的注释 --- */
  }
  
  .page-links-item:hover .page-links-avatar-frame {
    transform: scale(1.1);
  }

  /* * 头像图片 (.page-links-img)
   * 负责填充相框，并保持自身比例
   */
  .page-links-avatar-frame .page-links-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    /* --- 优化 3 (可选): 统一圆角 --- */
    border-radius: 12px; 
  }

  /* * 文字内容区域 */
  .page-links-content {
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  /* 友链名称 */
  .page-links-content strong {
    font-size: 16px;
    font-weight: 600;
    margin: 0 0 6px 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--text-color, #333);
  }

  /* 友链描述 */
  .page-links-content span {
    font-size: 14px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--text-muted-color, #666);
  }
  
  /* * “幽灵”链接 (Stretched Link)
   * 创建一个透明的可点击层，覆盖整个卡片
   */
  .page-links-item .stretched-link::after {
    content: "";
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 1;
    pointer-events: auto;
    /* --- 优化 4: 使用更简洁的透明写法 --- */
    background-color: transparent;
  }
</style>

<div class="page-links">
  
  {% for friend in site.data.friends %}
    <div class="page-links-item">
      <div class="page-links-avatar-frame">
        <img class="page-links-img" src="{{ friend.avatar_url }}" alt="{{ friend.name }} avatar">
      </div>
      
      <div class="page-links-content">
        <strong>{{ friend.name }}</strong>
        <span>{{ friend.desc }}</span>
      </div>
      
      <a href="{{ friend.url }}" target="_blank" rel="noopener noreferrer" class="stretched-link" aria-label="Visit {{ friend.name }}"></a>
    </div>
  {% endfor %}

</div>


<script>
  // 等待页面的基本结构（DOM）加载完毕后执行
  document.addEventListener('DOMContentLoaded', function() {
    // 找到我们的友链容器
    const linksContainer = document.querySelector('.page-links');
    
    // 如果找到了这个容器
    if (linksContainer) {
      // 就在0.1秒后给它添加 'loaded' 类
      // 这个微小的延迟是为了确保所有样式都能应用上
      setTimeout(function() {
        linksContainer.classList.add('loaded');
      }, 100);
    }
  });
</script>

<noscript>
  <style>
    .page-links {
      visibility: visible;
      opacity: 1;
    }
  </style>
</noscript>
