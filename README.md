# Vulcan - Enterprise-Grade DevOps Full-Stack Delivery Platform

Vulcan以罗马火神与锻造之神为名, 致力于成为企业软件交付的强大引擎。它打通从代码提交、持续集成、安全扫描、制品管理到自动化部署的完整链路, 提供稳定、高效的"锻造"流程。通过标准化、自动化的流水线, Vulcan帮助企业构筑坚固可靠的交付基石, 保障每一次发布都如神器出炉般精准可靠。

## 一、整体架构设计

### 1.1 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                     前端层 (Anvil)                          │
│  Vue 3 + TypeScript + Vite + Element Plus + Pinia           │
└─────────────────────────────────────────────────────────────┘
                            │ HTTP/WebSocket
┌─────────────────────────────────────────────────────────────┐
│                    API网关层 (可选)                         │
│              Nginx / Kong / Traefik                         │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│              后端服务层 (Vulcan)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  HTTP API    │  │  Task Queue  │  │  WebSocket   │       │
│  │   (Gin)      │  │  (Asynq)     │  │   (Gorilla)  │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  Auth Module │  │ Task Module  │  │  Hook Module │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────┐
│              工作流引擎 (Workflow Engine)               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ 状态机管理    │  │ 步骤调度器   │  │ 事件总线     │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                    数据层                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  MySQL   │  │ MongoDB  │  │  Redis   │  │  Nas     │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                  外部服务集成层                             │
│  GitLab │ Harbor │ Archery │ Lark │ Email                   │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 技术栈选型

#### 后端 (Golang)
- Web框架: Gin
- 任务队列: Asynq (替代 Celery)
- ORM: GORM
- 配置管理: Viper
- 日志: Zap
- 认证: JWT + Casbin
- 数据库驱动: MySQL, MongoDB, Redis
- HTTP客户端: Resty
- WebSocket: Gorilla WebSocket

#### 前端 (Vue 3)
- 框架: Vue 3 + TypeScript
- 构建工具: Vite
- UI组件: Element Plus
- 状态管理: Pinia
- 路由: Vue Router 4
- HTTP客户端: Axios
- 图表: ECharts

## 二、项目结构设计

### 2.1 后端项目结构 (vulcan)

```
vulcan/
├── cmd/
│   ├── api/              # HTTP API 服务入口
│   │   └── main.go
│   ├── worker/            # 异步任务 Worker 入口
│   │   └── main.go
│   └── migrate/           # 数据库迁移工具
│       └── main.go
├── internal/
│   ├── api/               # API 路由和处理器
│   │   ├── v1/
│   │   │   ├── auth.go    # 认证相关
│   │   │   ├── plan.go    # 版本计划
│   │   │   ├── task.go    # 任务管理
│   │   │   ├── system.go  # 系统管理
│   │   │   └── hook.go    # Webhook 处理
│   │   └── middleware/    # 中间件
│   │       ├── auth.go
│   │       ├── cors.go
│   │       └── logger.go
│   ├── service/           # 业务逻辑层
│   │   ├── auth/
│   │   │   ├── user.go
│   │   │   ├── role.go
│   │   │   └── permission.go
│   │   ├── plan/
│   │   │   ├── plan.go
│   │   │   └── version.go
│   │   ├── task/
│   │   │   ├── executor.go      # 任务执行器
│   │   │   ├── chain.go         # 任务链
│   │   │   ├── code_review.go   # 代码审查
│   │   │   ├── merge.go         # 代码合并
│   │   │   ├── tag.go           # Tag 管理
│   │   │   ├── image.go         # 镜像检查
│   │   │   ├── branch.go        # 分支推送
│   │   └── notification/
│   │       ├── lark.go
│   │       └── email.go
│   ├── repository/        # 数据访问层
│   │   ├── mysql/
│   │   │   ├── user.go
│   │   │   ├── plan.go
│   │   │   ├── task.go
│   │   │   └── system.go
│   │   ├── mongodb/
│   │   │   └── document.go
│   │   └── redis/
│   │       └── cache.go
│   ├── model/            # 数据模型
│   │   ├── user.go
│   │   ├── plan.go
│   │   ├── task.go
│   │   └── system.go
│   ├── worker/           # 异步任务定义
│   │   ├── code_review.go
│   │   ├── merge.go
│   │   ├── tag.go
│   │   ├── image.go
│   │   ├── branch.go
│   ├── client/           # 外部服务客户端
│   │   ├── gitlab/
│   │   ├── harbor/
│   │   ├── lark/
│   │   └── archery/
│   ├── config/           # 配置管理
│   │   └── config.go
│   ├── constant/         # 常量定义
│   │   └── constant.go
│   └── util/             # 工具函数
│       ├── logger.go
│       ├── validator.go
│       └── helper.go
├── pkg/                  # 可复用的公共包
│   ├── errors/
│   ├── response/
│   └── jwt/
├── configs/              # 配置文件
│   ├── config.yaml
│   └── config.dev.yaml
├── migrations/           # 数据库迁移文件
│   └── mysql/
├── scripts/              # 脚本文件
│   └── deploy.sh
├── docs/                 # 文档
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

### 2.2 前端项目结构 (Anvil)

```
Anvil/
├── src/
│   ├── api/              # API 接口定义
│   │   ├── auth.ts
│   │   ├── plan.ts
│   │   ├── task.ts
│   │   └── system.ts
│   ├── assets/           # 静态资源
│   │   ├── images/
│   │   └── styles/
│   ├── components/       # 公共组件
│   │   ├── common/
│   │   └── business/
│   ├── composables/      # 组合式函数
│   │   ├── useAuth.ts
│   │   └── useWebSocket.ts
│   ├── layouts/          # 布局组件
│   │   └── DefaultLayout.vue
│   ├── router/           # 路由配置
│   │   └── index.ts
│   ├── stores/           # Pinia 状态管理
│   │   ├── auth.ts
│   │   ├── plan.ts
│   │   └── task.ts
│   ├── views/            # 页面组件
│   │   ├── auth/
│   │   ├── plan/
│   │   │   ├── PlanList.vue
│   │   │   ├── PlanDetail.vue
│   │   │   └── PlanCreate.vue
│   │   ├── task/
│   │   │   ├── TaskList.vue
│   │   │   └── TaskDetail.vue
│   │   ├── system/
│   │   │   ├── SystemList.vue
│   │   │   └── AppManager.vue
│   │   └── user/
│   │       ├── UserList.vue
│   │       └── RoleManager.vue
│   ├── utils/            # 工具函数
│   │   ├── request.ts
│   │   ├── auth.ts
│   │   └── format.ts
│   ├── App.vue
│   └── main.ts
├── public/
├── .env.development
├── .env.production
├── vite.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

## 三、核心业务模块设计

### 3.1 任务执行流程设计

```go
// internal/service/task/executor.go

type TaskExecutor struct {
    planRepo    repository.PlanRepository
    taskRepo    repository.TaskRepository
    gitlabClient client.GitLabClient
    notifier    service.NotificationService
}

// 任务执行链
func (e *TaskExecutor) ExecuteTaskChain(ctx context.Context, planID int64) error {
    // 1. 获取计划信息
    plan, err := e.planRepo.GetByID(ctx, planID)
    if err != nil {
        return err
    }
    
    // 2. 根据当前步骤执行对应任务
    switch plan.CurrentStep {
    case constant.StepCodeReview:
        return e.executeCodeReview(ctx, plan)
    case constant.StepMergeRequest:
        return e.executeMergeRequest(ctx, plan)
    case constant.StepCreateTag:
        return e.executeCreateTag(ctx, plan)
    case constant.StepCheckImage:
        return e.executeCheckImage(ctx, plan)
    case constant.StepPushBranch:
        return e.executePushBranch(ctx, plan)
    // ... 其他步骤
    }
    
    return nil
}

// 代码审查
func (e *TaskExecutor) executeCodeReview(ctx context.Context, plan *model.Plan) error {
    // 调用 AI 代码审查服务
    // 更新任务状态
    // 发送通知
}

// 合并代码
func (e *TaskExecutor) executeMergeRequest(ctx context.Context, plan *model.Plan) error {
    // 创建 MR
    // 等待合并
    // 更新状态
}

// 创建 Tag
func (e *TaskExecutor) executeCreateTag(ctx context.Context, plan *model.Plan) error {
    // 创建 GitLab Tag
    // 触发 CI/CD
    // 更新状态
}

// 检查镜像
func (e *TaskExecutor) executeCheckImage(ctx context.Context, plan *model.Plan) error {
    // 检查 Harbor 镜像
    // 检查 NAS 数据包
    // 更新状态
}

// 推送分支
func (e *TaskExecutor) executePushBranch(ctx context.Context, plan *model.Plan) error {
    // 推送配置文件到 GitLab
    // 更新状态
}
```

### 3.2 任务状态机设计

```go
// internal/service/task/state_machine.go

type TaskStateMachine struct {
    transitions map[constant.Step]map[constant.Status]constant.Step
}

func NewTaskStateMachine() *TaskStateMachine {
    return &TaskStateMachine{
        transitions: map[constant.Step]map[constant.Status]constant.Step{
            constant.StepCodeReview: {
                constant.StatusSuccess: constant.StepMergeRequest,
                constant.StatusFailed:  constant.StepCodeReview, // 重试
            },
            constant.StepMergeRequest: {
                constant.StatusSuccess: constant.StepCreateTag,
                constant.StatusFailed:  constant.StepMergeRequest,
            },
            constant.StepCreateTag: {
                constant.StatusSuccess: constant.StepCheckImage,
                constant.StatusFailed:  constant.StepCreateTag,
            },
            // ... 其他状态转换
        },
    }
}

func (sm *TaskStateMachine) GetNextStep(currentStep constant.Step, status constant.Status) (constant.Step, bool) {
    nextSteps, ok := sm.transitions[currentStep]
    if !ok {
        return currentStep, false
    }
    nextStep, ok := nextSteps[status]
    return nextStep, ok
}
```

### 3.3 异步任务队列设计

```go
// internal/worker/code_review.go

func HandleCodeReview(ctx context.Context, t *asynq.Task) error {
    var payload CodeReviewPayload
    if err := json.Unmarshal(t.Payload(), &payload); err != nil {
        return err
    }
    
    // 1. 调用 AI 代码审查
    reviewResult, err := aiClient.ReviewCode(ctx, payload.RepoID, payload.Branch)
    if err != nil {
        return err
    }
    
    // 2. 更新任务状态
    taskRepo.UpdateStatus(ctx, payload.TaskID, reviewResult.Status)
    
    // 3. 发送通知
    notifier.SendCodeReviewNotification(ctx, payload.PlanID, reviewResult)
    
    return nil
}
```

## 四、数据库设计

### 4.1 核心表结构

```sql
-- 用户表
CREATE TABLE `users` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `username` VARCHAR(64) NOT NULL UNIQUE,
    `email` VARCHAR(128),
    `super_user` TINYINT DEFAULT 0,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 角色表
CREATE TABLE `roles` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `name` VARCHAR(64) NOT NULL UNIQUE,
    `description` VARCHAR(255),
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 权限表
CREATE TABLE `permissions` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `name` VARCHAR(64) NOT NULL,
    `uri` VARCHAR(255) NOT NULL,
    `method` VARCHAR(16) NOT NULL,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 版本计划表
CREATE TABLE `plans` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `version_no` VARCHAR(64) NOT NULL,
    `system_id` BIGINT NOT NULL,
    `software_type` VARCHAR(32),
    `state` INT NOT NULL DEFAULT 0,
    `current_step` INT NOT NULL DEFAULT 0,
    `creator_id` BIGINT NOT NULL,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX `idx_version_no` (`version_no`),
    INDEX `idx_system_id` (`system_id`)
);

-- 任务表
CREATE TABLE `tasks` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `plan_id` BIGINT NOT NULL,
    `step_id` INT NOT NULL,
    `status` INT NOT NULL DEFAULT 0,
    `run_count` INT DEFAULT 0,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX `idx_plan_id` (`plan_id`),
    INDEX `idx_step_id` (`step_id`)
);

-- 子任务表
CREATE TABLE `subtasks` (
    `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
    `task_id` BIGINT NOT NULL,
    `repo_id` BIGINT NOT NULL,
    `app_id` BIGINT NOT NULL,
    `status` INT NOT NULL DEFAULT 0,
    `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX `idx_task_id` (`task_id`),
    INDEX `idx_repo_id` (`repo_id`)
);
```

## 五、框架搭建步骤

### 5.1 后端初始化

```bash
# 1. 创建项目
mkdir vulcan && cd vulcan
go mod init github.com/dkhubs/vulcan

# 2. 安装依赖
go get github.com/gin-gonic/gin
go get github.com/hibiken/asynq
go get gorm.io/gorm
go get gorm.io/driver/mysql
go get go.mongodb.org/mongo-driver
go get github.com/go-redis/redis/v8
go get github.com/spf13/viper
go get go.uber.org/zap
go get github.com/golang-jwt/jwt/v4
go get github.com/casbin/casbin/v2

# 3. 创建目录结构
mkdir -p cmd/{api,worker,migrate}
mkdir -p internal/{api/v1,service,repository/{mysql,mongodb,redis},model,worker,client,config,constant,util}
mkdir -p pkg/{errors,response,jwt}
mkdir -p configs migrations/scripts
```

### 5.2 前端初始化

```bash
# 1. 创建项目
npm create vite@latest anvil -- --template vue-ts

# 2. 安装依赖
cd anvil
npm install
npm install element-plus @element-plus/icons-vue
npm install pinia vue-router@4 axios
npm install echarts
npm install -D sass

# 3. 创建目录结构
mkdir -p src/{api,assets,components/{common,business},composables,layouts,router,stores,views/{auth,plan,task,system,user},utils}
```

## 六、关键实现示例

### 6.1 后端 API 示例

```go
// internal/api/v1/plan.go

type PlanHandler struct {
    planService *service.PlanService
}

func (h *PlanHandler) CreatePlan(c *gin.Context) {
    var req CreatePlanRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.Error(c, err)
        return
    }
    
    plan, err := h.planService.CreatePlan(c.Request.Context(), &req)
    if err != nil {
        response.Error(c, err)
        return
    }
    
    response.Success(c, plan)
}

func (h *PlanHandler) ExecuteTask(c *gin.Context) {
    planID, _ := strconv.ParseInt(c.Param("id"), 10, 64)
    
    // 异步执行任务链
    task, err := asynq.NewTask("task:execute", payload)
    if err != nil {
        response.Error(c, err)
        return
    }
    
    client.Enqueue(task)
    response.Success(c, "任务已提交")
}
```

### 6.2 前端 API 调用示例

```typescript
// src/api/plan.ts

import request from '@/utils/request'

export interface Plan {
  id: number
  versionNo: string
  systemId: number
  state: number
  currentStep: number
}

export const planApi = {
  // 获取计划列表
  getPlanList(params: { page: number; size: number }) {
    return request.get<{ list: Plan[]; total: number }>('/api/v1/plans', { params })
  },
  
  // 创建计划
  createPlan(data: CreatePlanRequest) {
    return request.post<Plan>('/api/v1/plans', data)
  },
  
  // 执行任务
  executeTask(planId: number) {
    return request.post(`/api/v1/plans/${planId}/execute`)
  }
}
```

### 6.3 WebSocket 实时更新

```go
// internal/api/v1/websocket.go

func (h *WebSocketHandler) HandleTaskStatus(c *gin.Context) {
    conn, err := upgrader.Upgrade(c.Writer, c.Request, nil)
    if err != nil {
        return
    }
    defer conn.Close()
    
    for {
        // 发送任务状态更新
        status := taskService.GetTaskStatus(planID)
        conn.WriteJSON(status)
        time.Sleep(1 * time.Second)
    }
}
```

## 七、部署方案

### 7.1 Docker 部署

```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o vulcan ./cmd/api

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/vulcan .
COPY --from=builder /app/configs ./configs
CMD ["./vulcan"]
```

## 八、迁移计划

### 8.1 数据迁移
1. 导出现有 MySQL 数据
2. 转换数据格式（如需要）
3. 导入到新数据库
4. 验证数据完整性

### 8.2 功能迁移优先级
1. 认证授权模块（高优先级）
2. 版本计划管理（高优先级）
3. 任务执行链（核心功能）
4. 外部服务集成（GitLab、Harbor）
5. 通知系统（飞书、邮件）
6. 前端界面重构

## 九、优化建议

### 9.1 性能优化
- 使用 Redis 缓存热点数据
- 数据库查询优化（索引、分页）
- 异步任务队列处理耗时操作
- WebSocket 推送实时状态

### 9.2 可维护性
- 清晰的模块划分
- 统一的错误处理
- 完善的日志记录
- 单元测试和集成测试

### 9.3 安全性
- JWT 认证
- RBAC 权限控制
- API 限流
- 输入验证和 SQL 注入防护
