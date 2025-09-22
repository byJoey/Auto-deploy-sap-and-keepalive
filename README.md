# 部署失败请等到早上八点在部署 百分之百可以 
### 前置要求
* GitHub 账户：需要有一个 GitHub 账户来创建仓库和设置工作流
* SAP Cloud Foundry 账户：需要有 SAP Cloud Foundry 的有效账户，点此注册：https://www.sap.com

## 部署步骤

1. Fork本仓库

2. 在Actions菜单允许 `I understand my workflows, go ahead and enable them` 按钮

3. 在 GitHub 仓库中设置以下 secrets（Settings → Secrets and variables → Actions → New repository secret）：
   - `EMAIL`: Cloud Foundry账户邮箱
   - `PASSWORD`: Cloud Foundry账户密码
   
   **注意：新版本工作流已自动检测组织和空间，无需再设置以下secrets：**
   - ~~`SG_ORG`: 新加坡组织名称~~（已自动获取）
   - ~~`US_ORG`: 美国组织名称~~（已自动获取）
   - ~~`SPACE`: Cloud Foundry空间名称~~（已自动获取）
   - `部署的时候支持支持选择直连`: 打钩即可

4. **设置Docker容器环境变量(也是在secrets里设置)**
   
   使用固定隧道token部署，请在cloudflare里设置端口为8001：
   
   **设置基础环境变量：**
   - `UUID`：节点uuid，如果开启了哪吒v1，部署完一个之后一定要修改UUID，否则agent会被覆盖
   - `ARGO_DOMAIN`：固定隧道域名，未设置将使用临时隧道
   - `ARGO_AUTH`：固定隧道json或token，未设置将使用临时隧道
   - `SUB_PATH`：订阅token，未设置默认是sub
   
   **可选环境变量：**
   - `NEZHA_SERVER`：v1形式: nezha.xxx.com:8008  v0形式：nezha.xxx.com
   - `NEZHA_PORT`：V1哪吒没有这个
   - `NEZHA_KEY`：v1的NZ_CLIENT_SECRET或v0的agent密钥
   - `CFIP`：优选域名或优选ip 
   - `CFPORT`：优选域名或优选ip对应端口 
   - `CHAT_ID`：Telegram聊天ID（可选）
   - `BOT_TOKEN`：Telegram机器人令牌（可选）
  
5. **开始部署**
   * 在GitHub仓库的Actions页面找到"自动部署SAP"工作流
   * 点击"Run workflow"按钮
   * 根据需要选择或填写以下参数：
     - environment: 选择部署环境（staging/production）
     - region: 选择部署区域（SG/US）
     - app_name: （可选）指定应用名称，留空则自动生成
   * 点击绿色的"Run workflow"按钮开始部署
   
   **工作流会自动：**
   - 检测并选择第一个可用的组织
   - 检测并选择第一个可用的空间
   - 显示部署目标信息
   - 完成应用部署和配置

6. **获取节点信息**
   * 点开运行的actions，查看Deploy application步骤的日志
   * 在日志末尾可以看到完整的部署信息，包括：
     - 应用名称
     - 访问域名
     - 部署组织和空间
     - 部署区域
   * 订阅地址：域名/$SUB_PATH（SUB_PATH变量没设置默认是sub，即订阅为：域名/sub）

## 应用管理

### 删除应用
* 在Actions页面找到"删除指定区域所有SAP应用"工作流
* 点击"Run workflow"按钮
* 选择要删除应用的区域（SG或US）
* 在确认框中输入"确认删除"
* 点击"Run workflow"开始删除
* 工作流会自动检测组织和空间，并删除该区域下的所有应用

## 保活机制

### GitHub Actions自动保活（推荐）
* 本仓库包含"自动保活SAP"工作流，会自动重启所有区域的应用
* 默认每天tuc 23:00执行（可在工作流文件中调整cron时间）
* 工作流会自动检测所有区域的组织和空间
* 并行处理SG和US区域的应用重启
* actions保活可能存在时间误差，建议根据前两天的情况进行适当调整cron时间


## 新版本优势

### 简化配置
- 无需手动设置组织和空间secrets
- 自动检测并使用第一个可用的组织和空间
- 减少了配置错误的可能性

### 智能部署
- 自动生成应用名称（可自定义）
- 显示详细的部署目标信息
- 提供完整的部署结果反馈

### 统一管理
- 所有工作流都使用相同的自动检测逻辑
- 支持多区域并行操作
- 提供详细的操作日志和统计信息

## 注意事项

1. **必需配置**：确保EMAIL和PASSWORD这两个GitHub Secrets已正确配置
2. **区域权限**：多区域部署需先开通权限，确保目标区域有足够的内存配额
3. **安全设置**：建议设置SUB_PATH订阅token，防止节点泄露
4. **首次使用**：第一次使用时工作流会自动检测可用资源，请查看运行日志确认检测结果
5. **应用命名**：如果不指定应用名称，系统会自动生成格式为"区域代码+随机字符串"的名称
6. **环境变量**：可以通过.env文件设置额外的环境变量，工作流会自动读取并应用
