---
creation date: 2025-03-11 19:45
modification date: 2025-03-11 21:07
tags:
  - git
  - github
  - CI/CD
---
GitHub Actions 是 GitHub 提供的一款 **CI/CD（持续集成/持续部署）工具**，可以帮助我们自动**构建、测试、编译、打包、部署**项目

可以用来自动化打包、部署到 github pages（[Github王炸功能Pages,免费免服务器上线网站,详细教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV12H4y1N7Q4?spm_id_from=333.788.videopod.sections&vd_source=dbefe8621d153f1b1aaef0768a993d25)） 

## 核心思想与流程
以拍电影的视角解读：
不同的演员有不同的剧本，不同的剧本中各步骤是顺序的，不同演员之间可以并行表演。演员的共同配合完成电影的拍摄。

两个文件夹 `.github/workflow` 和一个文件 `workflow.yml` (只要是 yml 格式的文件即可)
Github Actions 的核心是“什么时候干什么事”
-  `on` 关键字定义工作流发生的时机
-  `jobs` 定义具体的工作，可以有多个 `job` 并行
	- `steps` 定义每个 `job` 内部的执行流程
		- `uses` 可以定义引用的工作流（`actions/checkout@v4` 可以在服务器上自动切换到仓库目录，[Marketplace](https://github.com/marketplace?type=actions) 有很多预定义的 github actions）
		- `run` 可以在终端执行命令
		- `with` 提供引用工作流所需的参数
		- `env` 使用仓库中定义的环境变量或秘钥
	- `runs-on` 定义运行在什么机器上（可以使用本地服务器也可以使用 Github 官方提供的服务器）
	- `permissions` 定义权限

## 示例
```yml
name: Repository Dependents Tracker

on:
  workflow_dispatch:
    inputs:
      target_repo:
        description: 'Target repository (owner/name)'
        required: true
        default: 'langchain-ai/langgraph'

permissions: read-all

concurrency:
  group: ${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  track-dependents:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Derive parameters
        id: vars
        run: |
          REPO_SLUG=$(echo "${{ inputs.target_repo || 'langchain-ai/langgraph' }}" | sed 's/[^a-zA-Z0-9]/-/g')
          echo "slug=${REPO_SLUG}" >> $GITHUB_OUTPUT
          echo "docfile=docs/${REPO_SLUG}-dependents.md" >> $GITHUB_OUTPUT

      - name: Analyze Dependents
        uses: nvuillam/github-dependents-info@v1.5.1
        with:
          repo: ${{ inputs.target_repo || 'langchain-ai/langgraph' }}
          markdownfile: ${{ steps.vars.outputs.docfile }}
          badgemarkdownfile: "README.md"
          # sort: "stars"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Prepare commit
        run: sudo chown -R $USER:$USER .

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          branch: dependents-update-${{ steps.vars.outputs.slug }}
          commit-message: "📊 Update ${{ inputs.target_repo }} dependents report"
          title: "📈 ${{ inputs.target_repo }} Ecosystem Usage Report"
          body: "Automated dependency tracking for ${{ inputs.target_repo }}"
```
