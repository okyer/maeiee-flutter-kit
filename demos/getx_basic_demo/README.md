# getx_basic_demo

基于 Flutter 开发的“待办事项”项目，支持添加事项、删除事项、事项列表、事项详情。

## 一、项目结构总览

```
getx_basic_demo/
├── lib/
│   ├── main.dart                 # 应用入口，全局配置
│   └── app/
│       ├── models/               # 数据模型层
│       │   └── todo.dart         # Todo实体类
│       ├── services/             # 业务逻辑层（全局状态）
│       │   └── todo_service.dart # Todo数据服务（生命周期与App一致）
│       ├── controllers/          # 控制器层（页面状态）
│       │   ├── home_controller.dart    # 首页控制器 + Binding
│       │   └── detail_controller.dart  # 详情页控制器 + Binding
│       ├── views/                # 视图层
│       │   ├── pages/            # 页面
│       │   │   ├── home_page.dart
│       │   │   └── detail_page.dart
│       │   └── widgets/          # 可复用组件
│       │       └── todo_list_item.dart
│       └── routes/               # 路由配置
│           └── app_routes.dart   # 路由表 + GetPage配置
```

## 二、组件依赖关系图

```
┌─────────────────────────────────────────────────────────────┐
│                         main.dart                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │               GetMaterialApp (全局根)                 │   │
│  │  • initialBinding: AppBinding (注入 TodoService)      │   │
│  │  • getPages: AppPages.pages (路由表)                  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  Routes/Router │   │  TodoService    │   │  Controllers    │
│  (路由管理层)   │   │  (全局服务层)    │   │  (页面控制层)    │
│               │   │                 │   │                 │
│ • Routes.HOME │   │ • todos (状态)  │   │ • HomeController │
│ • Routes.DETAIL│   │ • filter (状态) │   │ • DetailController│
│ • GetPage配置 │   │ • 业务方法      │   │                 │
│               │   │   (CRUD操作)    │   │ 每个Controller  │
│               │   │                 │   │ 包含对应的Binding│
└───────────────┘   └─────────────────┘   └─────────────────┘
        │                     ▲                     │
        │                     │                     │
        ▼                     │                     ▼
┌───────────────┐             │             ┌─────────────────┐
│    Pages      │◄────────────┴────────────►│    Models       │
│  (UI页面层)    │       读取/修改状态        │  (数据模型层)    │
│               │                           │                 │
│ • HomePage    │                           │ • Todo          │
│ • DetailPage  │                           │ • toJson/fromJson│
│               │                           │                 │
│ 通过 Get.find │                           │ 纯数据结构，无逻辑│
│ 获取Controller│                           │                 │
└───────────────┘                           └─────────────────┘
        │
        ▼
┌───────────────┐
│    Widgets    │
│  (可复用组件)   │
│               │
│ • TodoListItem│
│               │
│ 通过参数接收   │
│ 数据和回调    │
└───────────────┘
```

## 三、数据流向图

```
用户操作 (UI)
    │
    ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Widget    │───►│  Controller │───►│   Service   │
│  (按钮点击等) │    │  (处理交互逻辑)│    │  (管理全局状态)│
└─────────────┘    └─────────────┘    └─────────────┘
                                              │
                         ┌────────────────────┘
                         ▼
                    ┌─────────────┐
                    │    Model    │
                    │  (数据实体)  │
                    └─────────────┘
                         │
                         ▼
                    Rx 响应式更新
                         │
    ┌────────────────────┼────────────────────┐
    ▼                    ▼                    ▼
┌─────────┐      ┌─────────────┐      ┌─────────────┐
│  Obx()  │      │   Obx()     │      │   Obx()     │
│ (首页列表)│      │  (详情页)    │      │  (其他页面)  │
└─────────┘      └─────────────┘      └─────────────┘
```

## 四、各层职责详解

| 层级 | 文件/类 | 职责 | 生命周期 |
|------|---------|------|----------|
| **Entry** | `main.dart` `AppBinding` | 应用入口，注入全局服务 | App |
| **Routes** | `app_routes.dart` | 集中管理路由，配置页面和Binding | App |
| **Services** | `todo_service.dart` | 全局状态管理，业务逻辑，数据持久化 | App (`GetxService`) |
| **Controllers** | `*_controller.dart` | 页面状态，处理用户交互，调用Service | Page (`GetxController`) |
| **Models** | `todo.dart` | 数据实体，JSON序列化 | 数据对象 |
| **Views** | `*_page.dart` | 构建UI，监听状态变化，无业务逻辑 | Widget |
| **Widgets** | `*_widget.dart` | 可复用UI组件，纯展示/交互 | Widget |

## 五、框架模板（推荐的项目结构）

基于以上分析，推荐的标准 GetX 项目模板：

```
lib/
├── main.dart                    # 应用入口
├── app/
│   ├── core/                    # 核心基础设施（可选）
│   │   ├── constants/           # 常量定义
│   │   ├── theme/               # 主题配置
│   │   └── utils/               # 工具函数
│   │
│   ├── models/                  # 数据模型
│   │   ├── user_model.dart
│   │   └── todo_model.dart
│   │
│   ├── repositories/            # 数据源（来源本地数据库或者API调用，可选）
│   │   ├── user_repo.dart
│   │   └── todo_repo.dart
│   │
│   ├── controllers/             # 控制器（当前页面的业务逻辑和状态，使用Binding依赖注入）
│   │   ├── user_controler.dart
│   │   └── todo_controler.dart
│   │
│   ├── views/                   # 页面和组件
│   │   ├── pages/               # 页面
│   │   │   ├── user_page.dart
│   │   │   └── todo_page.dart
│   │   │
│   │   └── widgets/             # 可复用组件
│   │       ├── user_item.dart
│   │       └── todo_list_item.dart
│   │
│   ├── routes/                  # 路由管理
│   │   └── app_routes.dart      # 路由表 + GetPage配置
│   │
│   └── services/                # 全局服务（跨页面状态共享、复杂业务逻辑等）
│       ├── auth_service.dart    # 认证服务
│       └── api_service.dart     # 网络请求
│
└── assets/                      # 静态资源
    ├── images/
    └── fonts/
```

## 六、关键设计原则

### 1. 依赖方向
```
View → Controller → Service → Model
     ↑（通过GetX依赖注入反向获取）
```

### 2. 状态管理分级
- **全局状态**（跨页面共享）→ 放在 `Service`（`GetxService`）
- **页面状态**（单个页面内）→ 放在 `Controller`（`GetxController`）
- **局部状态**（单个Widget）→ 使用 `StatefulWidget` 或 `Obx`

### 3. 响应式更新机制
```dart
// Service 中定义响应式数据
final todos = <Todo>[].obs;

// View 中监听变化
Obx(() => ListView(...todos...))
```

### 4. 依赖注入时机
```dart
// App 级别：在 AppBinding 中注入
Get.put(TodoService());  // 整个App生命周期

// Page 级别：在 Page Binding 中注入
Get.put(HomeController());  // 页面存在期间

// 或使用 LazyPut 延迟注入
Get.lazyPut(() => ApiService());
```

## 七、代码规范建议

| 规范 | 说明 |
|------|------|
| 文件名 | 小写 + 下划线：`home_controller.dart` |
| 类名 | 大驼峰：`HomeController` |
| Binding | 与 Controller 同名，放在同一文件或单独文件 |
| 路由常量 | 全大写：`static const HOME = '/home'` |
| 响应式变量 | 添加 `.obs` 后缀，类型明确：`final count = 0.obs` |
| 控制器释放 | 覆写 `onClose()` 释放资源（TextController等） |

这个架构模板可以用于后续的 Flutter 项目开发，具有良好的可扩展性和维护性。

