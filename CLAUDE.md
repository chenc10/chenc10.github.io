# 主页维护指南

本仓库是 Chen Chen（陈晨，上海交通大学）个人学术主页 https://chenc10.github.io 的源码，基于 [al-folio](https://github.com/alshedivat/al-folio)（Jekyll）模板，托管于 GitHub Pages。本文件记录目录结构、内容写法约定、发布流程与已知问题，供维护者（人或自动化助手）在改动前阅读。

## 1. 站点概况

- **线上地址**：https://chenc10.github.io
- **源码分支**：`master`。push 到 `master` 后，GitHub Actions 工作流 `Deploy site`（`.github/workflows/deploy.yml`）自动执行 `jekyll build`，并把 `_site/` 发布到 `gh-pages` 分支，通常 1–3 分钟后线上生效。
- **触发条件**：工作流只在改动 `assets/**`、`*.bib`、`*.md`、`*.yml`、`*.html`、`*.liquid`、`*.js`、`Gemfile` 时触发；`README.md` / `FAQ.md` / `INSTALL.md` / `CONTRIBUTING.md` 的改动不会触发部署。
- **页面**：About（首页 `/`，`_pages/about.md`）、Publications（`/publications/`，由 `_bibliography/papers.bib` 经 jekyll-scholar 生成，分 **Preprints**（`abbr={Arxiv}` 的条目，平铺不分年）与 **Refereed Papers**（其余条目，按年分组）两栏，见 `_pages/publications.md` 里的两个 `{% bibliography --query ... %}`）、Team（`/team/`，`_pages/Team.md`）、News 全量页（`/news/`，不在导航栏；首页只显示最近 12 条）。
- **未启用的模板功能**：博客（`_posts/`）、CV 页（`_data/cv.yml`）、Projects、Repositories、首页精选论文区块（`selected_papers: false`）。这些位置的内容是模板自带样例，不需要维护，也不要删除。

## 2. 目录速查

| 路径 | 内容 | 日常改动 |
| --- | --- | --- |
| `_news/announcement_YY-M-D.md` | 一条 News 一个文件 | 常改 |
| `_bibliography/papers.bib` | 全部论文，Publications 页由此生成 | 常改 |
| `assets/pdf/` | 论文 PDF，命名 `YYYY_venue_shortname.pdf` | 常改 |
| `_pages/Team.md` | 团队页（手写 HTML 卡片） | 偶尔 |
| `assets/img/` | 头像 `chen.jpg` / `chen_small.jpg` 与成员照片（拼音全名小写，如 `wangtianze.jpg`；`default.jpg` 为占位图） | 偶尔 |
| `_pages/about.md` | 首页个人简介；frontmatter 中的 `profile.more_info` 是办公室与邮箱 | 偶尔 |
| `_layouts/about.liquid` | 首页的 **Teaching / Honors / Professional Services** 三段写死在这个布局文件里（不在 about.md）；课程按课程名合并、年份列在后面；论文奖也要同步加进 Honors | 偶尔 |
| `_data/coauthorss.yml` | 合作者主页链接（见"已知问题"） | 偶尔 |
| `_config.yml` | 站点配置：`announcements.limit: 12`（首页 News 条数）、`scholar`（bib 渲染）、`exclude` 列表 | 仅在明确要求时 |
| `_layouts/`、`_includes/`、`_sass/`、`_plugins/`、`assets/css`、`assets/js` | 模板机制 | 不动 |
| `.github/workflows/` | CI 与部署 | 不动 |
| `_posts/`、`_data/cv.yml`、`_data/repositories.yml`、`_data/venues.yml`、`assets/img/publication_preview/` | 模板样例，未使用 | 不动 |

## 3. 内容约定

### 3.1 News（`_news/`）

- 文件名 `announcement_YY-M-D.md`：年取两位，月、日不补零，例如 `announcement_26-9-23.md`。同一天多条时可加后缀（如 `announcement_26-9-23-2.md`），显示顺序只看 frontmatter 里的 `date`。
- frontmatter 固定为：

  ```yaml
  ---
  layout: post
  date: 2026-9-23 07:59:00-0400
  inline: true
  related_posts: false
  ---
  ```

- 正文一句英文，不加标题。惯用句式：
  - 学生一作论文录用："One paper on <topic> is accepted by <VENUE YEAR>. Congratulations to <Name>!"
  - 有系统名或合作论文："The <System> work, which <one-line description>, is accepted by <VENUE YEAR>."（合作论文不写 Congratulations）
  - 获奖、基金、arXiv 发布等同样用一句话说清。
- 首页 News 区块按 `date` 倒序显示**全部**条目，放在一个 520px 高（默认可见约 10 条）、可滚动的框里（`_config.yml` 中 `announcements.scrollable: true`、`limit` 留空；高度写在 `_includes/news.liquid`，细滚动条样式在 `_sass/_base.scss` 末尾）；全量列表页仍是 `/news/`。

### 3.2 Publications（`_bibliography/papers.bib`）

- **新条目插在文件顶部**：位于开头两行 `---` 与 `@string{...}` 之后。页面按 `year` 倒序分组，同一年内按文件中的先后顺序显示。
- 必填字段：`abbr`（venue 缩写，如 `ASPLOS`、`OSDI`、`TPDS`、`Arxiv`，显示为左侧标签）、`title`、`author`、`booktitle`（会议）或 `journal`（期刊）、`year`。
- `abbr={Arxiv}` 的条目会自动进入页面顶部的 Preprints 栏，其余进入 Refereed Papers；所以 arXiv 预印本的 `abbr` 必须严格写 `Arxiv`。
- `author` 写 `Last, First and Last, First ...`；**通讯作者在姓后加星号**，如 `Chen*, Chen`，模板会显示星号。
- `year` 按**会议召开年份**填写（2026 年录用、2027 年召开的会议写 `2027`）。
- `booktitle` 用全称，如 `Proceedings of the ACM International Conference on Architectural Support for Programming Languages and Operating Systems`；期刊条目用 `journal={ACM Transactions on Architecture and Code Optimization}`，可附 `volume` / `number` / `pages`。
- `abstract={...}`：一段纯文本摘要，页面上显示为 ABS 按钮。特殊字符需转义（`\%`、`\&`、`\$`、`\#`、`\_`），不要含花括号和反斜杠命令。**新条目尽量补摘要**，来源优先级：出版社页面（IEEE Xplore 需用浏览器打开）> CVF/arXiv > 站内 PDF 抽取（需人工核对起止）。
- 常用可选字段：
  - `pdf={2026_asplos_impeller.pdf}`：只写文件名，文件放在 `assets/pdf/`。
  - `award={...}` 与 `award_name={:trophy: Best Paper Award}`：获奖说明，`award` 里可以写 Markdown 链接。
  - `video={URL}`、`html={URL}`、`code={URL}`、`slides={文件名}`、`poster={文件名}`、`arxiv={id}`、`doi={...}`。
  - `selected={true}`：精选标记（首页目前未开启精选区块）。
- arXiv 预印本写成 `@article`，`abbr={Arxiv}`，`journal={arXiv preprint arXiv:2605.18710}`；**正式录用后删除 arXiv 条目**，改为会议 / 期刊条目并保留 `pdf`。
- bib key 形如 `lastnameYEARkeyword`（如 `yu2026impeller`），全文件内唯一。
- 改完做最基本的语法自查：每条以 `}` 结束、字段之间有逗号、花括号配对。

### 3.3 Team（`_pages/Team.md`）

- 手写 HTML，分组依次为 Faculty / Ph.D. Students / Master Students / Undergraduate Students / Students Previously Mentored。
- 每位成员一个 `col-lg-6` 卡片（两列布局）：照片 `/assets/img/<拼音全名>.jpg`（暂无照片用 `default.jpg`）、姓名加粗、入学时间（如 `Joined in 2023 Fall`）、本科院校（如 `B.S. Sichuan University`）。新增成员复制相邻卡片改内容即可；毕业或离组的成员移到最后一组，按原样式补去向。
- 照片尽量接近正方形，控制在 200 KB 以内。

### 3.4 About（`_pages/about.md`）

- 正文为 Markdown。frontmatter 中 `subtitle` 是职称，`profile.image` 是头像文件名，`profile.more_info` 是办公室与邮箱；`news: true` 控制首页是否显示 News 区块。

## 4. 发布流程

```bash
git pull --ff-only
# ... 修改文件 ...
git add <具体文件>            # 不用 git add -A，避免带入无关文件
git commit -m "add ASPLOS'27 Impeller to publications and news"
git push origin master
```

- commit message 一句话说明改了什么即可（仓库历史惯用简短英文）。**不要附加 Co-Authored-By 等任何署名、尾注或生成标记。**
- push 后可在 https://github.com/chenc10/chenc10.github.io/actions 查看 `Deploy site` 是否成功；成功后 1–3 分钟线上生效，浏览器有缓存时强制刷新。同时被触发的 `Prettier code formatter` 与 `Check for broken links` 两个检查是模板自带的，历史上每次提交都失败，不影响发布，可忽略。
- 构建失败最常见的原因：bib 语法错误（缺逗号、括号不配对）、frontmatter YAML 错误、文件名含空格或中文。
- 禁止 `push --force`、rebase 已推送的历史、删除或改写他人的提交；拿不准的改动先确认。

## 5. 本地预览（可选）

- 推荐 Docker：仓库根目录执行 `docker compose up -d`（镜像 `amirpourmand/al-folio:latest`），浏览 http://localhost:8080 ，改动会自动重建。
  - 该镜像（2026-08 起）的 Ruby 为 4.0，`logger` 等库不再默认加载，容器自带的 `jekyll serve` 会直接报 LoadError。不要为此改仓库的 Gemfile，而是在容器内用覆盖用的 Gemfile 启动：

    ```bash
    docker compose exec -T jekyll bash -c 'printf "eval_gemfile \"/srv/jekyll/Gemfile\"\ngem \"logger\"\ngem \"csv\"\ngem \"base64\"\ngem \"bigdecimal\"\ngem \"ostruct\"\ngem \"observer\"\ngem \"benchmark\"\ngem \"drb\"\ngem \"mutex_m\"\n" > /tmp/Gemfile.preview; cp Gemfile.lock /tmp/Gemfile.preview.lock; export BUNDLE_GEMFILE=/tmp/Gemfile.preview; bundle install --quiet; bundle exec jekyll serve -d /tmp/_site --host 0.0.0.0 --port 8080 --force_polling'
    ```

  - 容器启动时的 `bundle install` 会改写仓库中**受 git 跟踪的 `Gemfile.lock`**，预览结束后务必 `git checkout -- Gemfile.lock`，绝不能把它带进提交（线上构建用的是 Ruby 3.2）。
- 不用 Docker 时可以只做静态检查，然后依赖 GitHub Actions 的构建结果：

  ```bash
  ruby -ryaml -e 'YAML.load_file("_config.yml"); puts "config ok"'
  for f in _news/*.md; do ruby -ryaml -e 'YAML.load(File.read(ARGV[0]).split("---")[1])' "$f" >/dev/null || echo "bad frontmatter: $f"; done
  python3 -c "s=open('_bibliography/papers.bib').read(); assert s.count('{')==s.count('}'), 'unbalanced braces'; print('bib braces ok')"
  ```

## 6. 已知问题与待办

- 尚无摘要的条目：`yu2026impeller`（ASPLOS'27，未公开）、`qiang2026fluxzk`（SC'26，未公开）；正式发表后补上。
- `_bibliography/papers.bib` 中两条 IWQoS'24 条目共用 key `zuo2024pas`，需要改名区分。
- News 提到但 bib 尚未收录的论文：SC'26（ZKP 合作论文）、ICPP'26、2026 年 5 月的 TACO 与两篇 IEEE JCC。
- 模板读取的是 `_data/coauthors.yml`（当前为空文件），而合作者链接实际写在 `_data/coauthorss.yml`（多一个 s），因此论文页的合作者超链接目前未生效；是否合并两个文件需要确认。

## 7. 注意事项

- 仓库是公开的，任何提交都会公开：不要放入未公开的稿件、内部资料、凭据，或联系方式以外的个人隐私。
- 本文件已加入 `_config.yml` 的 `exclude` 列表，不会被发布到线上站点。
