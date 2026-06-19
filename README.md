 Cloudflare Pages 和 BPB 面板搭建个人专属免费VPN 
https://www.haoyep.com/posts/cf-bpb-vpn/

1 搭建思路

当前 Cloudflare 收紧了 BPB 等项目的审查，直接使用源码或者原作者提供的混淆代码，很容易出现1101的报错。合理推测 Cloudflare 通过以下方面做了限制：

    代理类关键词：如 vless
    项目类关键词：如 bpb
    源码：如某个代理类代码被多次使用（这也是为什么混淆代码刚开始好使，过两三天又会出现1101的原因）

混淆代码可以绕过 Cloudflare 的审查，前提是使用同一份混淆代码的人不多。BPB 项目现在也提供了未混淆加密前的源代码，我们可以加密该代码来获得自己独一无二的混淆代码，从而成功完成搭建。

注意

如果你已经有成功搭建且运行很长时间的 BPB，不要轻易更新 _worker.js！能用就不要动！如果想体验新版本 BPB，可以重新创建个 worker。


2 前提

    Github 账号：通过 Github action 自动拉取最新源代码并进行混淆
    Cloudflare 账号
    域名：解决 Cloudflare Pages 自带域名被墙

    内网端口
    使用 vercel 加速 Github 图床
    通过 Cloudflare 和 jsDelivr 免费加速博客 GitHub 图床等静态资源
    Windows下使用hugo和Github Pages配置博客
    通过热点共享网络，让局域网内的手机等设备共享本机代理

利用 Cloudflare Pages 和 BPB 面板搭建个人专属免费VPN订阅节点, 避免 1101 等报错
Leehow Leehow 收录于 技术
2025-01-19 2026-06-17 约 2600 字 预计阅读 6 分钟 36124 次阅读  
https://cdn.haoyep.com/gh/leegical/Blog_img/cdnimg/202408290144107.png

无需域名，无需 SSL，通过 Cloudflare 和 BPB Panel，解决1101等报错，搭建一个个人专属永久免费且高速的免费 VPN。结合 Cloudflare 实现优选订阅永久免费节点订阅，为使用（singbox-core 和 xray-core）的跨平台客户端提供配置。
1 搭建思路

当前 Cloudflare 收紧了 BPB 等项目的审查，直接使用源码或者原作者提供的混淆代码，很容易出现1101的报错。合理推测 Cloudflare 通过以下方面做了限制：

    代理类关键词：如 vless
    项目类关键词：如 bpb
    源码：如某个代理类代码被多次使用（这也是为什么混淆代码刚开始好使，过两三天又会出现1101的原因）

混淆代码可以绕过 Cloudflare 的审查，前提是使用同一份混淆代码的人不多。BPB 项目现在也提供了未混淆加密前的源代码，我们可以加密该代码来获得自己独一无二的混淆代码，从而成功完成搭建。

注意

如果你已经有成功搭建且运行很长时间的 BPB，不要轻易更新 _worker.js！能用就不要动！如果想体验新版本 BPB，可以重新创建个 worker。
2 前提

    Github 账号：通过 Github action 自动拉取最新源代码并进行混淆
    Cloudflare 账号
    域名：解决 Cloudflare Pages 自带域名被墙

3 混淆代码
新建一个 Github 仓库，在仓库里新建文件夹 .github/workflows，并在该文件夹中创建 Obfuscate.yml 文件，粘贴代码如下（也可以去我提供的 demoPanel示例仓库 中复制相关代码。）：


name: Build Obfuscate BPB Panel

on:
  push:
    branches:
      - main
  schedule:
        # Runs everyday at 1:00 AM
        - cron: "0 1 * * *"

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out the code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "latest"

      - name: Install dependencies
        run: |
          npm install -g javascript-obfuscator

      - name: Clone BPB workjs
        run: |
          wget -O origin.js https://raw.githubusercontent.com/bia-pain-bache/BPB-Worker-Panel/refs/heads/main/build/unobfuscated-worker.js
      
      - name: Obfuscate BPB worker js
        run: |
          javascript-obfuscator origin.js --output _worker.js \
          --compact true \
          --control-flow-flattening true \
          --control-flow-flattening-threshold 1 \
          --dead-code-injection true \
          --dead-code-injection-threshold 1 \
          --identifier-names-generator hexadecimal \
          --rename-globals true \
          --string-array true \
          --string-array-encoding 'rc4' \
          --string-array-threshold 1 \
          --transform-object-keys true \
          --unicode-escape-sequence true

      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          branch: main
          commit_message: ':arrow_up: update latest bpb panel'
          commit_author: 'github-actions[bot] <github-actions[bot]@users.noreply.github.com>'
          push_options: '--set-upstream'

注意

仓库主分支名需为 main。

Obfuscate.yml 为你的代码仓库创建了一个 action，它将在每次 main 分支有 push 时、每天1点钟下载最新的 BPB 源代码，并执行混淆。

push Obfuscate.yml 到你的代码仓库。稍等片刻，仓库根目录中会出现两个新的文件：

    origin.js：最新未加密的 BPB 源代码
    _worker.js：混淆后的个人专属 BPB 代码

下载混淆后的代码文件 _worker.js 到本地，压缩为 worker.zip.

4 创建 Pages


