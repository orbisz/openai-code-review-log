评审日志存储仓库

基于GLM 4.5-air模型（可自由选择大模型）的代码自动评审组件，针对最近两次提交到GitHub上的代码的差异进行评审。
用户提交代码时触发评审，调用基于RAG优化的大模型分析代码差异并生成评审建议，评审结果通过微信公众号推送，评审日志持久化至独立仓库。该系统通过自动化评审辅助人工评审，显著提高了评审的效率与准确性；

>jar包存放在该日志仓库的release中；

用户可以通过在提交代码仓库的 github actions 配置 workflow 来使用该组件，如下：
```yaml
name: Build and Run OpenAiCodeReview By Main Remote Jar

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 2

      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'adopt'
          java-version: '11'

      - name: Create libs directory
        run: mkdir -p ./libs

      - name: Download openai-code-review-sdk JAR
        run: curl -L -o ./libs/openai-code-review-sdk-1.0.jar https://github.com/orbisz/openai-code-review-log/releases/download/v1.0/openai-code-review-sdk-1.0.jar

      - name: Get repository name
        id: repo-name
        run: echo "REPO_NAME=${GITHUB_REPOSITORY##*/}" >> $GITHUB_ENV

      - name: Get branch name
        id: branch-name
        run: echo "BRANCH_NAME=${GITHUB_REF#refs/heads/}" >> $GITHUB_ENV

      - name: Get commit author
        id: commit-author
        run: echo "COMMIT_AUTHOR=$(git log -1 --pretty=format:'%an <%ae>')" >> $GITHUB_ENV

      - name: Get commit message
        id: commit-message
        run: echo "COMMIT_MESSAGE=$(git log -1 --pretty=format:'%s')" >> $GITHUB_ENV

      - name: Print repository, branch name, commit author, and commit message
        run: |
          echo "Repository name is ${{ env.REPO_NAME }}"
          echo "Branch name is ${{ env.BRANCH_NAME }}"
          echo "Commit author is ${{ env.COMMIT_AUTHOR }}"
          echo "Commit message is ${{ env.COMMIT_MESSAGE }}" 

      - name: Run Code Review
        env:
          # Github 配置；GITHUB_REVIEW_LOG_URI「https://github.com/orbisz/openai-code-review-log」、GITHUB_TOKEN「https://github.com/settings/tokens」
          GITHUB_REVIEW_LOG_URI: ${{ secrets.CODE_REVIEW_LOG_URI }}
          GITHUB_TOKEN: ${{ secrets.CODE_TOKEN }}
          COMMIT_PROJECT: ${{ env.REPO_NAME }}
          COMMIT_BRANCH: ${{ env.BRANCH_NAME }}
          COMMIT_AUTHOR: ${{ env.COMMIT_AUTHOR }}
          COMMIT_MESSAGE: ${{ env.COMMIT_MESSAGE }}
          # 微信配置 「https://mp.weixin.qq.com/debug/cgi-bin/sandboxinfo?action=showinfo&t=sandbox/index」
          WEIXIN_APPID: ${{ secrets.WEIXIN_APPID }}
          WEIXIN_SECRET: ${{ secrets.WEIXIN_SECRET }}
          WEIXIN_TOUSER: ${{ secrets.WEIXIN_TOUSER }}
          WEIXIN_TEMPLATE_ID: ${{ secrets.WEIXIN_TEMPLATE_ID }}
          # OpenAi - ChatGLM 配置「https://open.bigmodel.cn/api/paas/v4/chat/completions」、「https://open.bigmodel.cn/usercenter/apikeys」
          CHATGLM_APIHOST: ${{ secrets.CHATGLM_APIHOST }}
          CHATGLM_APIKEYSECRET: ${{ secrets.CHATGLM_APIKEYSECRET }}


        run: java -jar ./libs/openai-code-review-sdk-1.0.jar
```
>环境变量需要配置在代码提交仓库的Repository secret中。

详细开发日志可见:https://orbisz.github.io/blogs/Project/code-review/code-review.html
