# 机器人调试工具合集

<div class="tools-hero">
  <p class="hero-subtitle">自主移动机器人开发 · 上位机调试套件</p>
  <p class="hero-desc">以下工具均经过验证，点击卡片即可获取最新版本安装包。</p>
</div>

---

<!--
  📌 添加新软件步骤：
  1. 复制下面任意一张卡片的 HTML 块
  2. 修改 emoji、名称、版本号、描述、标签
  3. 在 <a> 标签上设置：
     - data-repo="仓库所有者/仓库名"  例：hbrobot/HB-VmxPiTool
     - data-asset="文件名关键字"      例：Linux-x86_64.deb
  4. JS 会自动从该仓库最新 Release 中匹配对应资产并绑定下载链接
  5. 无需手动写完整 URL，发新版后自动指向最新版本
-->

<div class="tool-cards">

<!-- ========== 软件 1：HB-VmxPiTool ========== -->
<div class="tool-card">
  <div class="tool-card-inner">
    <div class="tool-card-top">
      <div class="tool-icon-wrap">
        <span class="tool-icon">🛠</span>
      </div>
      <div class="tool-info">
        <h3 class="tool-name">HB-VmxPiTool</h3>
        <span class="tool-version-badge" data-version>检查更新中…</span>
      </div>
    </div>
    <p class="tool-desc">VMX-PI 控制器上位机调试工具，支持底盘参数配置、运动学标定与实时状态监控。</p>
    <div class="tool-tags">
      <span class="tag">Linux</span>
      <span class="tag">x86_64</span>
      <span class="tag file">.deb</span>
    </div>
    <a href="#"
       class="dl-btn gh-release"
       data-repo="hbrobot/HB-VmxPiTool"
       data-asset="Linux-x86_64.deb">
      <svg class="dl-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
      下载 Linux 版
    </a>
  </div>
</div>

<!-- ========== 软件 2：NavMap-Studio ========== -->
<div class="tool-card">
  <div class="tool-card-inner">
    <div class="tool-card-top">
      <div class="tool-icon-wrap accent">
        <span class="tool-icon">🗺</span>
      </div>
      <div class="tool-info">
        <h3 class="tool-name">NavMap-Studio</h3>
        <span class="tool-version-badge" data-version>检查更新中…</span>
      </div>
    </div>
    <p class="tool-desc">导航地图编辑与路径规划调试工具，支持建图、标注、轨迹回放与参数调优。</p>
    <div class="tool-tags">
      <span class="tag">Ubuntu 22.04</span>
      <span class="tag">Windows 10+</span>
      <span class="tag file">.deb / .exe</span>
    </div>
    <div class="dl-row">
      <a href="#"
         class="dl-btn outline gh-release"
         data-repo="YXZHC/NavMap-Studio"
         data-asset="ubuntu22.04-amd64.deb">
        <svg class="dl-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
        Linux
      </a>
      <a href="#"
         class="dl-btn outline gh-release"
         data-repo="YXZHC/NavMap-Studio"
         data-asset="Windows-x64-Setup.exe">
        <svg class="dl-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
        Windows
      </a>
    </div>
  </div>
</div>


<!-- ========== 软件 3：（复制此块扩展新软件） ========== -->
<!--
<div class="tool-card">
  <div class="tool-card-inner">
    <div class="tool-card-top">
      <div class="tool-icon-wrap">
        <span class="tool-icon">📦</span>
      </div>
      <div class="tool-info">
        <h3 class="tool-name">软件名称</h3>
        <span class="tool-version-badge" data-version>检查更新中…</span>
      </div>
    </div>
    <p class="tool-desc">一句话描述软件用途。</p>
    <div class="tool-tags">
      <span class="tag">平台</span>
      <span class="tag file">.后缀</span>
    </div>
    <a href="#"
       class="dl-btn gh-release"
       data-repo="所有者/仓库名"
       data-asset="文件名关键字">
      <svg class="dl-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
      下载
    </a>
  </div>
</div>
-->

</div>

---

<style>
/* ===== 顶部介绍区 ===== */
.tools-hero {
  text-align: center;
  padding: 1.5rem 0 0.5rem;
  margin-bottom: 1.5rem;
}

.hero-subtitle {
  font-size: 0.85rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--md-accent-fg-color);
  font-weight: 600;
  margin: 0 0 0.5rem;
}

.hero-desc {
  font-size: 0.95rem;
  color: var(--md-default-fg-color--light);
  margin: 0;
}

/* ===== 卡片网格 ===== */
.tool-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(360px, 1fr));
  gap: 1.75rem;
  margin: 2rem 0;
}

/* ===== 卡片本体 ===== */
.tool-card {
  border-radius: 16px;
  overflow: hidden;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), box-shadow 0.3s ease;
}

.tool-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.1);
}

.tool-card-inner {
  background: var(--md-default-bg-color);
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: 16px;
  padding: 1.75rem;
  height: 100%;
  display: flex;
  flex-direction: column;
  transition: border-color 0.3s ease;
}

.tool-card:hover .tool-card-inner {
  border-color: var(--md-accent-fg-color);
}

/* ===== 头部 ===== */
.tool-card-top {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.1rem;
}

.tool-icon-wrap {
  width: 52px;
  height: 52px;
  border-radius: 14px;
  background: var(--md-code-bg-color);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.tool-icon-wrap.accent {
  background: color-mix(in srgb, var(--md-accent-fg-color) 12%, transparent);
}

.tool-icon {
  font-size: 1.5rem;
  line-height: 1;
}

.tool-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  min-width: 0;
}

.tool-name {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.tool-version-badge {
  display: inline-block;
  font-size: 0.75rem;
  font-family: var(--md-code-font-family);
  color: var(--md-accent-fg-color);
  background: color-mix(in srgb, var(--md-accent-fg-color) 10%, transparent);
  padding: 0.15rem 0.55rem;
  border-radius: 6px;
  width: fit-content;
  font-weight: 500;
  white-space: nowrap;
}

/* ===== 描述 ===== */
.tool-desc {
  font-size: 0.9rem;
  color: var(--md-default-fg-color--light);
  line-height: 1.65;
  margin: 0 0 1.1rem;
  flex-grow: 1;
}

/* ===== 标签 ===== */
.tool-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 1.4rem;
}

.tag {
  font-size: 0.72rem;
  padding: 0.22rem 0.6rem;
  border-radius: 99px;
  background: var(--md-code-bg-color);
  color: var(--md-default-fg-color--light);
  font-family: var(--md-code-font-family);
  font-weight: 500;
  white-space: nowrap;
}

.tag.file {
  background: color-mix(in srgb, var(--md-accent-fg-color) 8%, transparent);
  color: var(--md-accent-fg-color);
}

/* ===== 下载按钮 ===== */
.dl-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  border-radius: 10px;
  background: var(--md-accent-fg-color);
  color: #fff !important;
  font-weight: 600;
  font-size: 0.9rem;
  text-decoration: none !important;
  transition: all 0.25s ease;
  border: none;
  cursor: pointer;
  white-space: nowrap;
}

.dl-btn:hover {
  filter: brightness(1.1);
  box-shadow: 0 4px 14px color-mix(in srgb, var(--md-accent-fg-color) 40%, transparent);
}

.dl-btn.outline {
  background: transparent;
  color: var(--md-default-fg-color) !important;
  border: 1.5px solid var(--md-default-fg-color--lighter);
}

.dl-btn.outline:hover {
  border-color: var(--md-accent-fg-color);
  color: var(--md-accent-fg-color) !important;
  background: transparent;
  box-shadow: none;
}

.dl-btn.is-disabled {
  opacity: 0.5;
  pointer-events: none;
}

.dl-icon {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
}

.dl-row {
  display: flex;
  gap: 0.75rem;
}

.dl-row .dl-btn {
  flex: 1 1 0;
  min-width: 0;
}

/* ===== 暗色模式 ===== */
[data-md-color-scheme="slate"] .tool-card:hover {
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.4);
}

/* ===== 移动端 ===== */
@media (max-width: 600px) {
  .tool-cards {
    grid-template-columns: 1fr;
    gap: 1.25rem;
  }
  .dl-row {
    flex-direction: column;
  }
}
</style>

<script>
(function () {
  // 按仓库缓存 release 数据，避免重复请求
  const releaseCache = {};

  async function fetchLatestRelease(repo) {
    if (releaseCache[repo]) return releaseCache[repo];
    const resp = await fetch(`https://api.github.com/repos/${repo}/releases/latest`, {
      headers: { 'Accept': 'application/vnd.github.v3+json' }
    });
    if (!resp.ok) throw new Error(`GitHub API ${resp.status}`);
    const data = await resp.json();
    releaseCache[repo] = data;
    return data;
  }

  // 根据关键字匹配资产文件名（大小写不敏感）
  function matchAsset(assets, keyword) {
    const kw = keyword.toLowerCase();
    return assets.find(a => a.name.toLowerCase().includes(kw));
  }

  async function initButtons() {
    const buttons = document.querySelectorAll('a.gh-release[data-repo]');
    // 按仓库分组，同一仓库只请求一次 API
    const repos = [...new Set([...buttons].map(b => b.dataset.repo))];
    const results = {};

    await Promise.all(repos.map(async repo => {
      try {
        results[repo] = await fetchLatestRelease(repo);
      } catch (e) {
        results[repo] = { error: e.message, tag_name: '未知', assets: [] };
      }
    }));
    
    buttons.forEach(btn => {
      const repo = btn.dataset.repo;
      const assetKeyword = btn.dataset.asset;
      const release = results[repo];
    
      // 更新版本号 badge（同仓库的所有按钮共享一个版本）
      const card = btn.closest('.tool-card-inner');
      const badge = card && card.querySelector('[data-version]');
    
      if (release.error) {
        if (badge) badge.textContent = '获取失败';
        btn.classList.add('is-disabled');
        btn.addEventListener('click', e => e.preventDefault());
        return;
      }
    
      if (badge) badge.textContent = release.tag_name || '';
    
      const asset = matchAsset(release.assets || [], assetKeyword);
      if (asset) {
        btn.href = asset.browser_download_url;
        btn.setAttribute('target', '_blank');
        btn.setAttribute('rel', 'noopener');
      } else {
        btn.classList.add('is-disabled');
        btn.addEventListener('click', e => {
          e.preventDefault();
          alert(`未在最新 Release 中找到包含 "${assetKeyword}" 的安装包`);
        });
      }
    });
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initButtons);
  } else {
    initButtons();
  }
})();
</script>
