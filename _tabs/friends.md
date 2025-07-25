---
# the default layout is 'page'
icon: fas fa-link
order: 4
---
<style>
  .page-links-item {
    position: relative;
    display: flex;
    overflow: hidden;
    flex-direction: row;
    min-width: 0;
    height: var(--bs-card-height);
    color: var(--bs-body-color);
    word-wrap: break-word;
    background-color: var(--bs-card-bg);
    background-clip: border-box;
    border: var(--bs-card-border-width) solid var(--bs-card-border-color);
    border-radius: 10px;
    border-bottom: none !important;
  }
  .page-links-img {
    width: 45px;
    height: 45px;
    border-radius: 10px;
    transition: transform 0.3s ease;
  }
  .page-links-content {
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    overflow: hidden;
  }
  .page-links-item:hover .page-links-img {
    transform: scale(1.2);
  }
  .page-links-content span {
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>

<div class="page-links">
  <div class="row" id="showPlace"></div>
</div>

<script>
  // 关键改动：使用 Jekyll 的 `jsonify` 过滤器将 `_data/friends.yml` 的内容转换成 JavaScript 对象数组
  const list = {{ site.data.friends | jsonify }};

  // 随机排序函数
  function randomSort() {
    return 0.5 - Math.random();
  }
  const Links = list.sort(randomSort);

  // 获取要放置卡片的容器
  const showPlace = document.getElementById("showPlace");

  // 循环遍历朋友列表，为每个人创建卡片HTML
  // 注意：现在我们使用 friend.url, friend.name 等来访问数据，因为它们是对象了
  for (var i = 0; i < Links.length; i++) {
    var friend = Links[i];
    var cardHTML = `
      <div class="col-md-6 mb-4">
        <a class="post-preview page-links-item" href="${friend.url}" target="_blank">
          <div class="flex-shrink-0">
            <img class="page-links-img m-2" src="${friend.avatar}" alt="${friend.name}" />
          </div>
          <div class="flex-grow-1 ms-3 page-links-content">
            <h4 class="pt-0 my-2 mb-0 fw-bold">${friend.name}</h4>
            <span class="text-muted">${friend.desc}</span>
          </div>
        </a>
      </div>`;
    // 将创建好的卡片HTML追加到容器中
    showPlace.innerHTML += cardHTML;
  }
</script>
