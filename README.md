# vprofile-github-actions CI/CD & Docker Deployment Project

---

## 專案簡介

vprofile-github-actions是一個基於 Java/Maven 的 Web 應用程式，使用 Tomcat 10 運行。
此repo展示完整的 CI/CD 流程，包括程式碼建置、測試、安全掃描，以及將 Docker 映像推送到 AWS ECR。

---

## 核心技術與工具

| 類別    | 技術/工具                  | 說明               |
| :---- | :--------------------- | :--------------- |
| 後端    | Java, Maven            | 核心程式語言與建構工具      |
| 應用伺服器 | Apache Tomcat 10       | 運行 Java Web 應用程式 |
| 容器化   | Docker, Docker Compose | 應用程式容器化與本地環境協調   |
| 部署    | Ansible                | 自動化伺服器配置和部署      |
| CI/CD | GitHub Actions         | 自動化工作流程          |
| 安全掃描  | Trivy                  | 檔案系統漏洞掃描         |
| 映像倉庫  | AWS ECR                | 儲存和管理 Docker 映像  |

---

## CI/CD 工作流程 (.github/workflows/main.yml)

* 觸發條件

  * Push 或 Pull Request 至 main 分支
  * 手動觸發 (workflow_dispatch)
  * 定時排程：週一至週五 UTC 14:10

* Jobs

  * Build：Maven install → 產生 WAR → 上傳 Artifact
  * Testing：main 分支執行 Unit Test 與 Checkstyle
  * Security_Scan：使用 Trivy 掃描安全漏洞
  * BUILD_AND_PUBLISH：依據前面三個 Job，main 分支成功後登入 AWS ECR → 建置 Docker 映像 → 推送 ECR (vprofile-app-image)

---

## Dockerfile 說明

* 位置：Docker-files/app/multistage/Dockerfile

* 多階段建置 (Multi-stage Build)

* BUILD_IMAGE

  * 使用 maven:3.9.9-eclipse-temurin-21-jammy
  * 執行 mvn install，生成 vprofile-v2.war

* Final Image

  * 使用 tomcat:10-jdk21
  * 清空預設 webapps
  * 複製 BUILD_IMAGE 產生的 vprofile-v2.war，改名為 ROOT.war 部署
  * 開放 8080 port
