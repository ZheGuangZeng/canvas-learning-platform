# 📚 Canvas学习平台开发 - 完整操作指南

## 🎯 项目概述

本指南将帮助您使用 **Flutter + Supabase + LiveKit** 技术栈，基于 CCPM 项目管理系统开发一个类似 Canvas 的现代化学习平台。

### 技术栈
- **前端**: Flutter 3.24+ (跨平台)
- **后端**: Supabase (PostgreSQL + Edge Functions)
- **实时通信**: LiveKit + Supabase Realtime
- **存储**: Supabase Storage
- **认证**: Supabase Auth
- **项目管理**: CCPM (Claude Code PM)

---

## 🚀 第一阶段：项目初始化 (第1-3天)

### 步骤1：初始化CCPM系统
```bash
# 初始化项目管理系统
/pm:init

# 创建项目上下文
/context:create

# 更新CLAUDE.md配置
/init include rules from .claude/CLAUDE.md
```

**预期结果**: CCPM系统完成配置，GitHub CLI认证成功

### 步骤2：创建项目PRD
```bash
# 创建Canvas学习平台产品需求文档
/pm:prd-new canvas-learning-platform
```

**PRD包含内容**:
- 用户故事和需求
- 功能规格说明
- 技术约束条件
- 成功标准定义

### 步骤3：解析PRD并创建实施计划
```bash
# 将PRD转换为技术实施计划
/pm:prd-parse canvas-learning-platform

# 分解为具体任务
/pm:epic-decompose canvas-learning-platform

# 同步到GitHub
/pm:epic-sync canvas-learning-platform
```

**预期结果**: GitHub Issues创建完成，任务分解清晰

---

## 🛠️ 第二阶段：开发环境搭建 (第4-5天)

### 步骤4：Flutter环境配置

#### 4.1 检查Flutter环境
```bash
# 检查Flutter版本 (需要3.24+)
flutter doctor

# 如果版本过低，升级Flutter
flutter upgrade
```

#### 4.2 创建Flutter项目
```bash
# 创建新的Flutter项目
flutter create canvas_learning_platform
cd canvas_learning_platform

# 清理示例代码
rm lib/main.dart
rm test/widget_test.dart
```

#### 4.3 添加核心依赖
```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  
  # 后端服务
  supabase_flutter: ^2.5.6
  
  # 实时视频通话
  livekit_client: ^2.1.0
  
  # 状态管理
  provider: ^6.1.2
  riverpod: ^2.4.9
  
  # 路由导航
  go_router: ^13.2.1
  
  # UI组件
  cached_network_image: ^3.3.1
  image_picker: ^1.0.7
  file_picker: ^8.0.0+1
  
  # 媒体播放
  video_player: ^2.8.2
  audio_players: ^6.0.0
  
  # 权限管理
  permission_handler: ^11.3.0
  
  # 工具库
  uuid: ^4.3.3
  intl: ^0.19.0
  equatable: ^2.0.5

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  build_runner: ^2.4.9
  json_annotation: ^4.8.1
  json_serializable: ^6.7.1
```

#### 4.4 安装依赖
```bash
flutter pub get
flutter pub run build_runner build
```

### 步骤5：Supabase后端配置

#### 5.1 安装Supabase CLI
```bash
# macOS
brew install supabase/tap/supabase

# Windows (使用scoop)
scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
scoop install supabase

# 或使用npm
npm install -g supabase
```

#### 5.2 初始化Supabase项目
```bash
# 登录Supabase (需要访问令牌)
supabase login

# 在项目根目录初始化
supabase init

# 启动本地开发环境
supabase start
```

#### 5.3 创建Supabase云项目
1. 访问 https://supabase.com
2. 创建新项目
3. 获取项目URL和anon key
4. 配置本地环境变量

```bash
# .env
SUPABASE_URL=your-project-url
SUPABASE_ANON_KEY=your-anon-key
```

---

## 📊 第三阶段：数据库设计 (第6-8天)

### 步骤6：创建数据库架构

#### 6.1 用户管理架构
```sql
-- supabase/migrations/20240101000001_create_user_profiles.sql

-- 创建用户角色枚举
CREATE TYPE user_role AS ENUM ('student', 'teacher', 'admin');

-- 扩展用户配置表
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  avatar_url TEXT,
  role user_role NOT NULL DEFAULT 'student',
  bio TEXT,
  phone TEXT,
  date_of_birth DATE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 创建触发器自动更新时间戳
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

#### 6.2 课程管理架构
```sql
-- supabase/migrations/20240101000002_create_courses.sql

-- 课程状态枚举
CREATE TYPE course_status AS ENUM ('draft', 'published', 'archived');

-- 课程表
CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  description TEXT,
  instructor_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  thumbnail_url TEXT,
  status course_status NOT NULL DEFAULT 'draft',
  price DECIMAL(10,2) DEFAULT 0,
  duration_hours INTEGER DEFAULT 0,
  max_students INTEGER,
  start_date TIMESTAMP WITH TIME ZONE,
  end_date TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 课程分类表
CREATE TABLE course_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  description TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 课程分类关联表
CREATE TABLE course_category_relations (
  course_id UUID REFERENCES courses(id) ON DELETE CASCADE,
  category_id UUID REFERENCES course_categories(id) ON DELETE CASCADE,
  PRIMARY KEY (course_id, category_id)
);

-- 课程章节表
CREATE TABLE course_modules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  course_id UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  order_index INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 课程内容表
CREATE TABLE course_lessons (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  module_id UUID NOT NULL REFERENCES course_modules(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  content TEXT,
  video_url TEXT,
  duration_minutes INTEGER DEFAULT 0,
  order_index INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### 6.3 学习进度跟踪
```sql
-- supabase/migrations/20240101000003_create_learning_progress.sql

-- 课程注册表
CREATE TABLE enrollments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  course_id UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  enrolled_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  completed_at TIMESTAMP WITH TIME ZONE,
  progress_percentage INTEGER DEFAULT 0 CHECK (progress_percentage >= 0 AND progress_percentage <= 100),
  UNIQUE(student_id, course_id)
);

-- 课程完成进度
CREATE TABLE lesson_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  lesson_id UUID NOT NULL REFERENCES course_lessons(id) ON DELETE CASCADE,
  completed_at TIMESTAMP WITH TIME ZONE,
  watch_time_minutes INTEGER DEFAULT 0,
  UNIQUE(student_id, lesson_id)
);

-- 作业系统
CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  course_id UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  due_date TIMESTAMP WITH TIME ZONE,
  max_score INTEGER DEFAULT 100,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 作业提交
CREATE TABLE assignment_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID NOT NULL REFERENCES assignments(id) ON DELETE CASCADE,
  student_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  content TEXT,
  file_url TEXT,
  submitted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  score INTEGER,
  feedback TEXT,
  graded_at TIMESTAMP WITH TIME ZONE,
  graded_by UUID REFERENCES profiles(id),
  UNIQUE(assignment_id, student_id)
);
```

### 步骤7：设置行级安全策略 (RLS)

```sql
-- supabase/migrations/20240101000004_setup_rls_policies.sql

-- 启用所有表的RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE courses ENABLE ROW LEVEL SECURITY;
ALTER TABLE course_categories ENABLE ROW LEVEL SECURITY;
ALTER TABLE course_modules ENABLE ROW LEVEL SECURITY;
ALTER TABLE course_lessons ENABLE ROW LEVEL SECURITY;
ALTER TABLE enrollments ENABLE ROW LEVEL SECURITY;
ALTER TABLE lesson_progress ENABLE ROW LEVEL SECURITY;
ALTER TABLE assignments ENABLE ROW LEVEL SECURITY;
ALTER TABLE assignment_submissions ENABLE ROW LEVEL SECURITY;

-- 用户配置策略
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- 课程访问策略
CREATE POLICY "Anyone can view published courses" ON courses
  FOR SELECT USING (status = 'published');

CREATE POLICY "Teachers can manage own courses" ON courses
  FOR ALL USING (instructor_id = auth.uid());

-- 注册策略
CREATE POLICY "Students can view own enrollments" ON enrollments
  FOR SELECT USING (student_id = auth.uid());

CREATE POLICY "Students can enroll in courses" ON enrollments
  FOR INSERT WITH CHECK (student_id = auth.uid());

-- 进度跟踪策略
CREATE POLICY "Students can view own progress" ON lesson_progress
  FOR SELECT USING (student_id = auth.uid());

CREATE POLICY "Students can update own progress" ON lesson_progress
  FOR ALL USING (student_id = auth.uid());
```

### 步骤8：应用数据库迁移
```bash
# 应用所有迁移
supabase db push

# 生成类型定义
supabase gen types typescript --local > lib/types/database.dart
```

---

## 🎨 第四阶段：Flutter应用架构 (第9-12天)

### 步骤9：创建项目结构

```
lib/
├── main.dart                    # 应用入口
├── app.dart                     # App配置
├── core/                        # 核心功能
│   ├── constants/
│   │   ├── app_constants.dart
│   │   ├── api_endpoints.dart
│   │   └── ui_constants.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   ├── colors.dart
│   │   └── text_styles.dart
│   ├── utils/
│   │   ├── helpers.dart
│   │   ├── validators.dart
│   │   └── extensions.dart
│   └── errors/
│       └── exceptions.dart
├── features/                    # 功能模块
│   ├── auth/                   # 认证模块
│   │   ├── data/
│   │   ├── domain/
│   │   ├── presentation/
│   │   └── auth_module.dart
│   ├── dashboard/              # 仪表板
│   ├── courses/                # 课程管理
│   ├── chat/                   # 聊天功能
│   ├── video_call/            # 视频通话
│   └── profile/               # 用户配置
├── shared/                     # 共享组件
│   ├── widgets/
│   │   ├── common/
│   │   ├── forms/
│   │   └── media/
│   ├── services/
│   │   ├── supabase_service.dart
│   │   ├── livekit_service.dart
│   │   └── storage_service.dart
│   └── providers/
├── models/                     # 数据模型
│   ├── user_model.dart
│   ├── course_model.dart
│   └── message_model.dart
└── router/                     # 路由配置
    └── app_router.dart
```

### 步骤10：实现核心服务

#### 10.1 Supabase服务封装
```dart
// lib/shared/services/supabase_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';

class SupabaseService {
  static SupabaseService? _instance;
  static SupabaseService get instance => _instance ??= SupabaseService._();
  
  SupabaseService._();
  
  SupabaseClient get client => Supabase.instance.client;
  
  // 初始化Supabase
  static Future<void> initialize() async {
    await Supabase.initialize(
      url: 'YOUR_SUPABASE_URL',
      anonKey: 'YOUR_SUPABASE_ANON_KEY',
    );
  }
  
  // 认证相关方法
  User? get currentUser => client.auth.currentUser;
  
  Stream<AuthState> get authStateChanges => client.auth.onAuthStateChange;
  
  Future<AuthResponse> signInWithEmail(String email, String password) async {
    return await client.auth.signInWithPassword(
      email: email,
      password: password,
    );
  }
  
  Future<AuthResponse> signUp(String email, String password) async {
    return await client.auth.signUp(
      email: email,
      password: password,
    );
  }
  
  Future<void> signOut() async {
    await client.auth.signOut();
  }
}
```

#### 10.2 数据模型定义
```dart
// lib/models/user_model.dart
import 'package:equatable/equatable.dart';

enum UserRole { student, teacher, admin }

class UserProfile extends Equatable {
  final String id;
  final String email;
  final String fullName;
  final String? avatarUrl;
  final UserRole role;
  final String? bio;
  final String? phone;
  final DateTime? dateOfBirth;
  final DateTime createdAt;
  final DateTime updatedAt;

  const UserProfile({
    required this.id,
    required this.email,
    required this.fullName,
    this.avatarUrl,
    required this.role,
    this.bio,
    this.phone,
    this.dateOfBirth,
    required this.createdAt,
    required this.updatedAt,
  });

  factory UserProfile.fromJson(Map<String, dynamic> json) {
    return UserProfile(
      id: json['id'],
      email: json['email'],
      fullName: json['full_name'],
      avatarUrl: json['avatar_url'],
      role: UserRole.values.byName(json['role']),
      bio: json['bio'],
      phone: json['phone'],
      dateOfBirth: json['date_of_birth'] != null 
          ? DateTime.parse(json['date_of_birth'])
          : null,
      createdAt: DateTime.parse(json['created_at']),
      updatedAt: DateTime.parse(json['updated_at']),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'email': email,
      'full_name': fullName,
      'avatar_url': avatarUrl,
      'role': role.name,
      'bio': bio,
      'phone': phone,
      'date_of_birth': dateOfBirth?.toIso8601String(),
    };
  }

  @override
  List<Object?> get props => [
        id,
        email,
        fullName,
        avatarUrl,
        role,
        bio,
        phone,
        dateOfBirth,
        createdAt,
        updatedAt,
      ];
}
```

```dart
// lib/models/course_model.dart
enum CourseStatus { draft, published, archived }

class Course extends Equatable {
  final String id;
  final String title;
  final String? description;
  final String instructorId;
  final String? thumbnailUrl;
  final CourseStatus status;
  final double price;
  final int durationHours;
  final int? maxStudents;
  final DateTime? startDate;
  final DateTime? endDate;
  final DateTime createdAt;
  final DateTime updatedAt;

  const Course({
    required this.id,
    required this.title,
    this.description,
    required this.instructorId,
    this.thumbnailUrl,
    required this.status,
    required this.price,
    required this.durationHours,
    this.maxStudents,
    this.startDate,
    this.endDate,
    required this.createdAt,
    required this.updatedAt,
  });

  factory Course.fromJson(Map<String, dynamic> json) {
    return Course(
      id: json['id'],
      title: json['title'],
      description: json['description'],
      instructorId: json['instructor_id'],
      thumbnailUrl: json['thumbnail_url'],
      status: CourseStatus.values.byName(json['status']),
      price: (json['price'] ?? 0).toDouble(),
      durationHours: json['duration_hours'] ?? 0,
      maxStudents: json['max_students'],
      startDate: json['start_date'] != null 
          ? DateTime.parse(json['start_date'])
          : null,
      endDate: json['end_date'] != null 
          ? DateTime.parse(json['end_date'])
          : null,
      createdAt: DateTime.parse(json['created_at']),
      updatedAt: DateTime.parse(json['updated_at']),
    );
  }

  @override
  List<Object?> get props => [
        id,
        title,
        description,
        instructorId,
        thumbnailUrl,
        status,
        price,
        durationHours,
        maxStudents,
        startDate,
        endDate,
        createdAt,
        updatedAt,
      ];
}
```

---

## 🔐 第五阶段：认证系统实现 (第13-15天)

### 步骤11：认证功能实现

#### 11.1 认证状态管理
```dart
// lib/features/auth/presentation/providers/auth_provider.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

class AuthNotifier extends StateNotifier<AsyncValue<User?>> {
  AuthNotifier() : super(const AsyncValue.loading()) {
    _init();
  }

  final _supabase = Supabase.instance.client;

  void _init() {
    // 监听认证状态变化
    _supabase.auth.onAuthStateChange.listen((data) {
      state = AsyncValue.data(data.session?.user);
    });
    
    // 设置初始状态
    state = AsyncValue.data(_supabase.auth.currentUser);
  }

  Future<void> signInWithEmail(String email, String password) async {
    state = const AsyncValue.loading();
    
    try {
      final response = await _supabase.auth.signInWithPassword(
        email: email,
        password: password,
      );
      
      if (response.user != null) {
        state = AsyncValue.data(response.user);
      }
    } catch (e) {
      state = AsyncValue.error(e, StackTrace.current);
    }
  }

  Future<void> signUp(String email, String password, String fullName) async {
    state = const AsyncValue.loading();
    
    try {
      final response = await _supabase.auth.signUp(
        email: email,
        password: password,
        data: {'full_name': fullName},
      );
      
      if (response.user != null) {
        // 创建用户配置
        await _createUserProfile(response.user!, fullName);
        state = AsyncValue.data(response.user);
      }
    } catch (e) {
      state = AsyncValue.error(e, StackTrace.current);
    }
  }

  Future<void> _createUserProfile(User user, String fullName) async {
    await _supabase.from('profiles').insert({
      'id': user.id,
      'email': user.email!,
      'full_name': fullName,
      'role': 'student',
    });
  }

  Future<void> signOut() async {
    await _supabase.auth.signOut();
    state = const AsyncValue.data(null);
  }
}

final authProvider = StateNotifierProvider<AuthNotifier, AsyncValue<User?>>((ref) {
  return AuthNotifier();
});

// 当前用户配置Provider
final currentUserProfileProvider = FutureProvider<UserProfile?>((ref) async {
  final authState = ref.watch(authProvider);
  
  return authState.when(
    data: (user) async {
      if (user == null) return null;
      
      final response = await Supabase.instance.client
          .from('profiles')
          .select()
          .eq('id', user.id)
          .single();
      
      return UserProfile.fromJson(response);
    },
    loading: () => null,
    error: (_, __) => null,
  );
});
```

#### 11.2 登录页面
```dart
// lib/features/auth/presentation/pages/login_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

class LoginPage extends ConsumerStatefulWidget {
  const LoginPage({super.key});

  @override
  ConsumerState<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends ConsumerState<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isObscured = true;

  @override
  Widget build(BuildContext context) {
    final authState = ref.watch(authProvider);

    // 监听认证状态变化
    ref.listen<AsyncValue<User?>>(authProvider, (previous, next) {
      next.whenOrNull(
        data: (user) {
          if (user != null) {
            context.go('/dashboard');
          }
        },
        error: (error, _) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text('登录失败: $error')),
          );
        },
      );
    });

    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Form(
            key: _formKey,
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                // Logo和标题
                const Icon(
                  Icons.school,
                  size: 80,
                  color: Colors.blue,
                ),
                const SizedBox(height: 24),
                Text(
                  'Canvas学习平台',
                  style: Theme.of(context).textTheme.headlineMedium,
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 48),

                // 邮箱输入框
                TextFormField(
                  controller: _emailController,
                  keyboardType: TextInputType.emailAddress,
                  decoration: const InputDecoration(
                    labelText: '邮箱',
                    prefixIcon: Icon(Icons.email),
                    border: OutlineInputBorder(),
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return '请输入邮箱';
                    }
                    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                      return '请输入有效邮箱';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 16),

                // 密码输入框
                TextFormField(
                  controller: _passwordController,
                  obscureText: _isObscured,
                  decoration: InputDecoration(
                    labelText: '密码',
                    prefixIcon: const Icon(Icons.lock),
                    suffixIcon: IconButton(
                      icon: Icon(_isObscured ? Icons.visibility : Icons.visibility_off),
                      onPressed: () => setState(() => _isObscured = !_isObscured),
                    ),
                    border: const OutlineInputBorder(),
                  ),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return '请输入密码';
                    }
                    if (value.length < 6) {
                      return '密码至少6位';
                    }
                    return null;
                  },
                ),
                const SizedBox(height: 24),

                // 登录按钮
                ElevatedButton(
                  onPressed: authState.isLoading ? null : _handleLogin,
                  style: ElevatedButton.styleFrom(
                    padding: const EdgeInsets.symmetric(vertical: 16),
                  ),
                  child: authState.isLoading
                      ? const CircularProgressIndicator()
                      : const Text('登录'),
                ),
                const SizedBox(height: 16),

                // 注册链接
                TextButton(
                  onPressed: () => context.go('/register'),
                  child: const Text('还没有账号？立即注册'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }

  void _handleLogin() {
    if (_formKey.currentState!.validate()) {
      ref.read(authProvider.notifier).signInWithEmail(
            _emailController.text.trim(),
            _passwordController.text,
          );
    }
  }

  @override
  void dispose() {
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

### 步骤12：路由配置
```dart
// lib/router/app_router.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';
import 'package:supabase_flutter/supabase_flutter.dart';

final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    initialLocation: '/splash',
    redirect: (context, state) {
      final user = Supabase.instance.client.auth.currentUser;
      final isLoggedIn = user != null;
      
      // 认证保护的路由
      final protectedRoutes = ['/dashboard', '/courses', '/profile'];
      final isProtectedRoute = protectedRoutes.any((route) => 
          state.matchedLocation.startsWith(route));
      
      // 如果用户未登录且访问保护路由，重定向到登录页
      if (!isLoggedIn && isProtectedRoute) {
        return '/login';
      }
      
      // 如果用户已登录且访问认证页面，重定向到仪表板
      if (isLoggedIn && (state.matchedLocation == '/login' || 
                        state.matchedLocation == '/register')) {
        return '/dashboard';
      }
      
      return null;
    },
    routes: [
      GoRoute(
        path: '/splash',
        builder: (context, state) => const SplashPage(),
      ),
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: '/register',
        builder: (context, state) => const RegisterPage(),
      ),
      GoRoute(
        path: '/dashboard',
        builder: (context, state) => const DashboardPage(),
      ),
      GoRoute(
        path: '/courses',
        builder: (context, state) => const CoursesPage(),
      ),
      GoRoute(
        path: '/courses/:id',
        builder: (context, state) {
          final courseId = state.pathParameters['id']!;
          return CourseDetailPage(courseId: courseId);
        },
      ),
      GoRoute(
        path: '/profile',
        builder: (context, state) => const ProfilePage(),
      ),
    ],
  );
});
```

---

## 🎥 第六阶段：LiveKit视频通话集成 (第16-18天)

### 步骤13：LiveKit服务封装

#### 13.1 LiveKit服务类
```dart
// lib/shared/services/livekit_service.dart
import 'package:livekit_client/livekit_client.dart';
import 'package:flutter/foundation.dart';

class LiveKitService extends ChangeNotifier {
  static LiveKitService? _instance;
  static LiveKitService get instance => _instance ??= LiveKitService._();
  
  LiveKitService._();

  Room? _room;
  bool _isConnected = false;
  List<RemoteParticipant> _remoteParticipants = [];
  LocalParticipant? _localParticipant;

  // Getters
  Room? get room => _room;
  bool get isConnected => _isConnected;
  List<RemoteParticipant> get remoteParticipants => _remoteParticipants;
  LocalParticipant? get localParticipant => _localParticipant;

  // 连接到房间
  Future<void> connect({
    required String url,
    required String token,
    required String roomName,
  }) async {
    try {
      _room = Room(
        roomOptions: const RoomOptions(
          adaptiveStream: true,
          dynacast: true,
          defaultCameraCaptureOptions: CameraCaptureOptions(
            maxFrameRate: 30,
            params: VideoParametersPresets.h720_169,
          ),
        ),
      );

      // 监听房间事件
      _room!.addListener(_onRoomUpdate);

      // 连接到房间
      await _room!.connect(url, token);
      _isConnected = true;
      _localParticipant = _room!.localParticipant;

      notifyListeners();
      
      // 启用摄像头和麦克风
      await _enableCameraAndMicrophone();
      
    } catch (e) {
      debugPrint('LiveKit连接失败: $e');
      throw Exception('连接失败: $e');
    }
  }

  // 启用摄像头和麦克风
  Future<void> _enableCameraAndMicrophone() async {
    try {
      // 请求权限
      await _room!.localParticipant?.setCameraEnabled(true);
      await _room!.localParticipant?.setMicrophoneEnabled(true);
    } catch (e) {
      debugPrint('启用摄像头/麦克风失败: $e');
    }
  }

  // 房间状态更新
  void _onRoomUpdate() {
    _remoteParticipants = _room?.remoteParticipants.values.toList() ?? [];
    notifyListeners();
  }

  // 切换摄像头
  Future<void> toggleCamera() async {
    if (_localParticipant != null) {
      final enabled = _localParticipant!.isCameraEnabled();
      await _localParticipant!.setCameraEnabled(!enabled);
      notifyListeners();
    }
  }

  // 切换麦克风
  Future<void> toggleMicrophone() async {
    if (_localParticipant != null) {
      final enabled = _localParticipant!.isMicrophoneEnabled();
      await _localParticipant!.setMicrophoneEnabled(!enabled);
      notifyListeners();
    }
  }

  // 切换屏幕共享
  Future<void> toggleScreenShare() async {
    if (_localParticipant != null) {
      final enabled = _localParticipant!.isScreenShareEnabled();
      await _localParticipant!.setScreenShareEnabled(!enabled);
      notifyListeners();
    }
  }

  // 断开连接
  Future<void> disconnect() async {
    if (_room != null) {
      await _room!.disconnect();
      _room!.removeListener(_onRoomUpdate);
      _room = null;
      _isConnected = false;
      _remoteParticipants.clear();
      _localParticipant = null;
      notifyListeners();
    }
  }

  @override
  void dispose() {
    disconnect();
    super.dispose();
  }
}
```

#### 13.2 视频通话界面
```dart
// lib/features/video_call/presentation/pages/video_call_page.dart
import 'package:flutter/material.dart';
import 'package:livekit_client/livekit_client.dart';

class VideoCallPage extends StatefulWidget {
  final String roomName;
  final String token;
  final String serverUrl;

  const VideoCallPage({
    super.key,
    required this.roomName,
    required this.token,
    required this.serverUrl,
  });

  @override
  State<VideoCallPage> createState() => _VideoCallPageState();
}

class _VideoCallPageState extends State<VideoCallPage> {
  final LiveKitService _liveKitService = LiveKitService.instance;
  bool _isConnecting = true;

  @override
  void initState() {
    super.initState();
    _connectToRoom();
  }

  Future<void> _connectToRoom() async {
    try {
      await _liveKitService.connect(
        url: widget.serverUrl,
        token: widget.token,
        roomName: widget.roomName,
      );
      setState(() => _isConnecting = false);
    } catch (e) {
      setState(() => _isConnecting = false);
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('连接失败: $e')),
        );
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        title: Text('房间: ${widget.roomName}'),
        backgroundColor: Colors.black54,
        actions: [
          IconButton(
            icon: const Icon(Icons.call_end),
            onPressed: () {
              _liveKitService.disconnect();
              Navigator.pop(context);
            },
          ),
        ],
      ),
      body: _isConnecting
          ? const Center(child: CircularProgressIndicator())
          : AnimatedBuilder(
              animation: _liveKitService,
              builder: (context, child) {
                return Column(
                  children: [
                    // 主视频区域
                    Expanded(
                      flex: 3,
                      child: _buildMainVideo(),
                    ),
                    
                    // 远程参与者网格
                    if (_liveKitService.remoteParticipants.isNotEmpty)
                      Expanded(
                        flex: 1,
                        child: _buildParticipantGrid(),
                      ),
                    
                    // 控制面板
                    _buildControlPanel(),
                  ],
                );
              },
            ),
    );
  }

  Widget _buildMainVideo() {
    final localParticipant = _liveKitService.localParticipant;
    final localVideoTrack = localParticipant?.videoTracks.isNotEmpty == true
        ? localParticipant!.videoTracks.first.track as VideoTrack?
        : null;

    if (localVideoTrack != null) {
      return VideoTrackRenderer(
        localVideoTrack,
        fit: RTCVideoViewObjectFit.RTCVideoViewObjectFitCover,
      );
    }

    return Container(
      color: Colors.grey[800],
      child: const Center(
        child: Icon(
          Icons.videocam_off,
          size: 64,
          color: Colors.white54,
        ),
      ),
    );
  }

  Widget _buildParticipantGrid() {
    return Container(
      height: 120,
      padding: const EdgeInsets.all(8),
      child: ListView.builder(
        scrollDirection: Axis.horizontal,
        itemCount: _liveKitService.remoteParticipants.length,
        itemBuilder: (context, index) {
          final participant = _liveKitService.remoteParticipants[index];
          final videoTrack = participant.videoTracks.isNotEmpty
              ? participant.videoTracks.first.track as VideoTrack?
              : null;

          return Container(
            width: 80,
            margin: const EdgeInsets.only(right: 8),
            decoration: BoxDecoration(
              color: Colors.grey[800],
              borderRadius: BorderRadius.circular(8),
            ),
            child: videoTrack != null
                ? ClipRRect(
                    borderRadius: BorderRadius.circular(8),
                    child: VideoTrackRenderer(
                      videoTrack,
                      fit: RTCVideoViewObjectFit.RTCVideoViewObjectFitCover,
                    ),
                  )
                : Center(
                    child: Text(
                      participant.name ?? 'User',
                      style: const TextStyle(
                        color: Colors.white,
                        fontSize: 12,
                      ),
                    ),
                  ),
          );
        },
      ),
    );
  }

  Widget _buildControlPanel() {
    final localParticipant = _liveKitService.localParticipant;
    final isCameraEnabled = localParticipant?.isCameraEnabled() ?? false;
    final isMicEnabled = localParticipant?.isMicrophoneEnabled() ?? false;
    final isScreenShareEnabled = localParticipant?.isScreenShareEnabled() ?? false;

    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceEvenly,
        children: [
          // 摄像头开关
          FloatingActionButton(
            onPressed: _liveKitService.toggleCamera,
            backgroundColor: isCameraEnabled ? Colors.white : Colors.red,
            child: Icon(
              isCameraEnabled ? Icons.videocam : Icons.videocam_off,
              color: isCameraEnabled ? Colors.black : Colors.white,
            ),
          ),

          // 麦克风开关
          FloatingActionButton(
            onPressed: _liveKitService.toggleMicrophone,
            backgroundColor: isMicEnabled ? Colors.white : Colors.red,
            child: Icon(
              isMicEnabled ? Icons.mic : Icons.mic_off,
              color: isMicEnabled ? Colors.black : Colors.white,
            ),
          ),

          // 屏幕共享
          FloatingActionButton(
            onPressed: _liveKitService.toggleScreenShare,
            backgroundColor: isScreenShareEnabled ? Colors.blue : Colors.white,
            child: Icon(
              Icons.screen_share,
              color: isScreenShareEnabled ? Colors.white : Colors.black,
            ),
          ),

          // 挂断
          FloatingActionButton(
            onPressed: () {
              _liveKitService.disconnect();
              Navigator.pop(context);
            },
            backgroundColor: Colors.red,
            child: const Icon(Icons.call_end, color: Colors.white),
          ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _liveKitService.disconnect();
    super.dispose();
  }
}
```

---

## 💬 第七阶段：实时聊天系统 (第19-21天)

### 步骤14：聊天数据库设计

```sql
-- supabase/migrations/20240101000005_create_chat_system.sql

-- 聊天室表
CREATE TABLE chat_rooms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  course_id UUID REFERENCES courses(id) ON DELETE CASCADE,
  is_private BOOLEAN DEFAULT FALSE,
  created_by UUID NOT NULL REFERENCES profiles(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 聊天室成员表
CREATE TABLE chat_room_members (
  room_id UUID NOT NULL REFERENCES chat_rooms(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  role TEXT DEFAULT 'member' CHECK (role IN ('admin', 'moderator', 'member')),
  PRIMARY KEY (room_id, user_id)
);

-- 消息表
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  room_id UUID NOT NULL REFERENCES chat_rooms(id) ON DELETE CASCADE,
  sender_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  message_type TEXT DEFAULT 'text' CHECK (message_type IN ('text', 'image', 'file', 'video')),
  file_url TEXT,
  reply_to UUID REFERENCES messages(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

-- 消息状态表（已读状态）
CREATE TABLE message_status (
  message_id UUID NOT NULL REFERENCES messages(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  read_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  PRIMARY KEY (message_id, user_id)
);

-- 启用实时订阅
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE chat_room_members;

-- RLS策略
ALTER TABLE chat_rooms ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_room_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE message_status ENABLE ROW LEVEL SECURITY;

-- 聊天室权限策略
CREATE POLICY "Members can view chat rooms" ON chat_rooms
  FOR SELECT USING (
    id IN (
      SELECT room_id FROM chat_room_members WHERE user_id = auth.uid()
    )
  );

-- 消息权限策略
CREATE POLICY "Members can view messages" ON messages
  FOR SELECT USING (
    room_id IN (
      SELECT room_id FROM chat_room_members WHERE user_id = auth.uid()
    )
  );

CREATE POLICY "Members can send messages" ON messages
  FOR INSERT WITH CHECK (
    room_id IN (
      SELECT room_id FROM chat_room_members WHERE user_id = auth.uid()
    ) AND sender_id = auth.uid()
  );
```

### 步骤15：聊天服务实现

```dart
// lib/features/chat/data/chat_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';
import '../../../models/message_model.dart';
import '../../../models/chat_room_model.dart';

class ChatService {
  final SupabaseClient _supabase = Supabase.instance.client;

  // 获取用户聊天室列表
  Future<List<ChatRoom>> getUserChatRooms() async {
    final userId = _supabase.auth.currentUser!.id;
    
    final response = await _supabase
        .from('chat_rooms')
        .select('''
          *,
          chat_room_members!inner(user_id, role, joined_at)
        ''')
        .eq('chat_room_members.user_id', userId)
        .order('created_at', ascending: false);

    return response.map((json) => ChatRoom.fromJson(json)).toList();
  }

  // 创建聊天室
  Future<ChatRoom> createChatRoom({
    required String name,
    String? description,
    String? courseId,
    bool isPrivate = false,
  }) async {
    final userId = _supabase.auth.currentUser!.id;

    // 创建聊天室
    final roomResponse = await _supabase
        .from('chat_rooms')
        .insert({
          'name': name,
          'description': description,
          'course_id': courseId,
          'is_private': isPrivate,
          'created_by': userId,
        })
        .select()
        .single();

    // 将创建者加入聊天室
    await _supabase.from('chat_room_members').insert({
      'room_id': roomResponse['id'],
      'user_id': userId,
      'role': 'admin',
    });

    return ChatRoom.fromJson(roomResponse);
  }

  // 加入聊天室
  Future<void> joinChatRoom(String roomId) async {
    final userId = _supabase.auth.currentUser!.id;

    await _supabase.from('chat_room_members').insert({
      'room_id': roomId,
      'user_id': userId,
      'role': 'member',
    });
  }

  // 获取聊天室消息
  Stream<List<Message>> getChatRoomMessages(String roomId) {
    return _supabase
        .from('messages')
        .stream(primaryKey: ['id'])
        .eq('room_id', roomId)
        .order('created_at', ascending: true)
        .map((data) => data
            .where((json) => json['deleted_at'] == null)
            .map((json) => Message.fromJson(json))
            .toList());
  }

  // 发送消息
  Future<Message> sendMessage({
    required String roomId,
    required String content,
    String messageType = 'text',
    String? fileUrl,
    String? replyTo,
  }) async {
    final userId = _supabase.auth.currentUser!.id;

    final response = await _supabase
        .from('messages')
        .insert({
          'room_id': roomId,
          'sender_id': userId,
          'content': content,
          'message_type': messageType,
          'file_url': fileUrl,
          'reply_to': replyTo,
        })
        .select('''
          *,
          profiles:sender_id(id, full_name, avatar_url)
        ''')
        .single();

    return Message.fromJson(response);
  }

  // 标记消息为已读
  Future<void> markMessageAsRead(String messageId) async {
    final userId = _supabase.auth.currentUser!.id;

    await _supabase.from('message_status').upsert({
      'message_id': messageId,
      'user_id': userId,
    });
  }

  // 上传文件
  Future<String> uploadFile(String filePath, String fileName) async {
    final userId = _supabase.auth.currentUser!.id;
    final timestamp = DateTime.now().millisecondsSinceEpoch;
    final storageFilePath = 'chat/$userId/$timestamp-$fileName';

    await _supabase.storage
        .from('chat-files')
        .upload(storageFilePath, File(filePath));

    return _supabase.storage
        .from('chat-files')
        .getPublicUrl(storageFilePath);
  }
}
```

### 步骤16：聊天界面实现

```dart
// lib/features/chat/presentation/pages/chat_room_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../../models/message_model.dart';
import '../../../models/chat_room_model.dart';
import '../providers/chat_providers.dart';

class ChatRoomPage extends ConsumerStatefulWidget {
  final ChatRoom chatRoom;

  const ChatRoomPage({
    super.key,
    required this.chatRoom,
  });

  @override
  ConsumerState<ChatRoomPage> createState() => _ChatRoomPageState();
}

class _ChatRoomPageState extends ConsumerState<ChatRoomPage> {
  final TextEditingController _messageController = TextEditingController();
  final ScrollController _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    // 自动滚动到底部
    WidgetsBinding.instance.addPostFrameCallback((_) {
      _scrollToBottom();
    });
  }

  void _scrollToBottom() {
    if (_scrollController.hasClients) {
      _scrollController.animateTo(
        _scrollController.position.maxScrollExtent,
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeOut,
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    final messagesAsync = ref.watch(chatMessagesProvider(widget.chatRoom.id));

    return Scaffold(
      appBar: AppBar(
        title: Text(widget.chatRoom.name),
        actions: [
          IconButton(
            icon: const Icon(Icons.videocam),
            onPressed: () => _startVideoCall(),
          ),
          IconButton(
            icon: const Icon(Icons.info),
            onPressed: () => _showRoomInfo(),
          ),
        ],
      ),
      body: Column(
        children: [
          // 消息列表
          Expanded(
            child: messagesAsync.when(
              data: (messages) {
                // 监听消息变化，自动滚动到底部
                WidgetsBinding.instance.addPostFrameCallback((_) {
                  _scrollToBottom();
                });

                return ListView.builder(
                  controller: _scrollController,
                  padding: const EdgeInsets.all(16),
                  itemCount: messages.length,
                  itemBuilder: (context, index) {
                    final message = messages[index];
                    final isMe = message.senderId == 
                        Supabase.instance.client.auth.currentUser?.id;
                    
                    return _buildMessageBubble(message, isMe);
                  },
                );
              },
              loading: () => const Center(child: CircularProgressIndicator()),
              error: (error, _) => Center(
                child: Text('加载消息失败: $error'),
              ),
            ),
          ),

          // 消息输入框
          _buildMessageInput(),
        ],
      ),
    );
  }

  Widget _buildMessageBubble(Message message, bool isMe) {
    return Align(
      alignment: isMe ? Alignment.centerRight : Alignment.centerLeft,
      child: Container(
        margin: const EdgeInsets.symmetric(vertical: 4),
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
        constraints: BoxConstraints(
          maxWidth: MediaQuery.of(context).size.width * 0.7,
        ),
        decoration: BoxDecoration(
          color: isMe ? Colors.blue : Colors.grey[300],
          borderRadius: BorderRadius.circular(20),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          mainAxisSize: MainAxisSize.min,
          children: [
            // 发送者姓名（如果不是自己）
            if (!isMe) ...[
              Text(
                message.senderName ?? '未知用户',
                style: const TextStyle(
                  fontSize: 12,
                  fontWeight: FontWeight.bold,
                  color: Colors.black54,
                ),
              ),
              const SizedBox(height: 4),
            ],

            // 消息内容
            _buildMessageContent(message),

            // 时间戳
            const SizedBox(height: 4),
            Text(
              _formatTime(message.createdAt),
              style: TextStyle(
                fontSize: 10,
                color: isMe ? Colors.white70 : Colors.black54,
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildMessageContent(Message message) {
    switch (message.messageType) {
      case MessageType.text:
        return Text(
          message.content,
          style: TextStyle(
            color: message.senderId == 
                Supabase.instance.client.auth.currentUser?.id
                ? Colors.white
                : Colors.black87,
          ),
        );
      
      case MessageType.image:
        return Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            if (message.content.isNotEmpty) ...[
              Text(message.content),
              const SizedBox(height: 8),
            ],
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                message.fileUrl!,
                width: 200,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return const Icon(Icons.error);
                },
              ),
            ),
          ],
        );
      
      case MessageType.file:
        return Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Icon(Icons.attach_file, size: 16),
            const SizedBox(width: 8),
            Flexible(
              child: Text(
                message.content.isNotEmpty 
                    ? message.content
                    : '文件',
                overflow: TextOverflow.ellipsis,
              ),
            ),
          ],
        );
      
      default:
        return Text(message.content);
    }
  }

  Widget _buildMessageInput() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.grey[100],
        border: Border(top: BorderSide(color: Colors.grey[300]!)),
      ),
      child: Row(
        children: [
          // 附件按钮
          IconButton(
            icon: const Icon(Icons.attach_file),
            onPressed: _showAttachmentOptions,
          ),

          // 输入框
          Expanded(
            child: TextField(
              controller: _messageController,
              decoration: const InputDecoration(
                hintText: '输入消息...',
                border: OutlineInputBorder(),
                contentPadding: EdgeInsets.symmetric(
                  horizontal: 16,
                  vertical: 8,
                ),
              ),
              maxLines: null,
              textInputAction: TextInputAction.send,
              onSubmitted: (_) => _sendMessage(),
            ),
          ),

          const SizedBox(width: 8),

          // 发送按钮
          IconButton(
            icon: const Icon(Icons.send),
            onPressed: _sendMessage,
          ),
        ],
      ),
    );
  }

  void _sendMessage() {
    final content = _messageController.text.trim();
    if (content.isEmpty) return;

    ref.read(chatServiceProvider).sendMessage(
          roomId: widget.chatRoom.id,
          content: content,
        );

    _messageController.clear();
  }

  void _showAttachmentOptions() {
    showModalBottomSheet(
      context: context,
      builder: (context) => SafeArea(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            ListTile(
              leading: const Icon(Icons.photo),
              title: const Text('照片'),
              onTap: () {
                Navigator.pop(context);
                _pickImage();
              },
            ),
            ListTile(
              leading: const Icon(Icons.attach_file),
              title: const Text('文件'),
              onTap: () {
                Navigator.pop(context);
                _pickFile();
              },
            ),
          ],
        ),
      ),
    );
  }

  void _pickImage() {
    // 实现图片选择
    // 使用 image_picker 包
  }

  void _pickFile() {
    // 实现文件选择
    // 使用 file_picker 包
  }

  void _startVideoCall() {
    // 启动视频通话
    // 集成 LiveKit
  }

  void _showRoomInfo() {
    // 显示聊天室信息
  }

  String _formatTime(DateTime time) {
    final now = DateTime.now();
    final difference = now.difference(time);

    if (difference.inDays > 0) {
      return '${time.day}/${time.month}';
    } else if (difference.inHours > 0) {
      return '${time.hour}:${time.minute.toString().padLeft(2, '0')}';
    } else if (difference.inMinutes > 0) {
      return '${difference.inMinutes}分钟前';
    } else {
      return '刚刚';
    }
  }

  @override
  void dispose() {
    _messageController.dispose();
    _scrollController.dispose();
    super.dispose();
  }
}
```

---

## 🎓 第八阶段：课程管理系统 (第22-25天)

### 步骤17：课程管理功能实现

```dart
// lib/features/courses/data/course_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';
import '../../../models/course_model.dart';

class CourseService {
  final SupabaseClient _supabase = Supabase.instance.client;

  // 获取已发布课程列表
  Future<List<Course>> getPublishedCourses({
    int limit = 20,
    int offset = 0,
  }) async {
    final response = await _supabase
        .from('courses')
        .select('''
          *,
          profiles:instructor_id(full_name, avatar_url),
          enrollments(count)
        ''')
        .eq('status', 'published')
        .range(offset, offset + limit - 1)
        .order('created_at', ascending: false);

    return response.map((json) => Course.fromJson(json)).toList();
  }

  // 获取用户创建的课程
  Future<List<Course>> getInstructorCourses() async {
    final userId = _supabase.auth.currentUser!.id;
    
    final response = await _supabase
        .from('courses')
        .select('''
          *,
          enrollments(count)
        ''')
        .eq('instructor_id', userId)
        .order('created_at', ascending: false);

    return response.map((json) => Course.fromJson(json)).toList();
  }

  // 获取用户注册的课程
  Future<List<Course>> getEnrolledCourses() async {
    final userId = _supabase.auth.currentUser!.id;
    
    final response = await _supabase
        .from('enrollments')
        .select('''
          *,
          courses:course_id(
            *,
            profiles:instructor_id(full_name, avatar_url)
          )
        ''')
        .eq('student_id', userId)
        .order('enrolled_at', ascending: false);

    return response
        .map((json) => Course.fromJson(json['courses']))
        .toList();
  }

  // 创建课程
  Future<Course> createCourse({
    required String title,
    String? description,
    String? thumbnailUrl,
    double price = 0,
    int durationHours = 0,
    int? maxStudents,
    DateTime? startDate,
    DateTime? endDate,
  }) async {
    final userId = _supabase.auth.currentUser!.id;

    final response = await _supabase
        .from('courses')
        .insert({
          'title': title,
          'description': description,
          'instructor_id': userId,
          'thumbnail_url': thumbnailUrl,
          'price': price,
          'duration_hours': durationHours,
          'max_students': maxStudents,
          'start_date': startDate?.toIso8601String(),
          'end_date': endDate?.toIso8601String(),
          'status': 'draft',
        })
        .select()
        .single();

    return Course.fromJson(response);
  }

  // 更新课程
  Future<Course> updateCourse(String courseId, Map<String, dynamic> updates) async {
    updates['updated_at'] = DateTime.now().toIso8601String();

    final response = await _supabase
        .from('courses')
        .update(updates)
        .eq('id', courseId)
        .select()
        .single();

    return Course.fromJson(response);
  }

  // 发布课程
  Future<void> publishCourse(String courseId) async {
    await _supabase
        .from('courses')
        .update({'status': 'published'})
        .eq('id', courseId);
  }

  // 注册课程
  Future<void> enrollCourse(String courseId) async {
    final userId = _supabase.auth.currentUser!.id;

    await _supabase.from('enrollments').insert({
      'student_id': userId,
      'course_id': courseId,
    });
  }

  // 取消注册
  Future<void> unenrollCourse(String courseId) async {
    final userId = _supabase.auth.currentUser!.id;

    await _supabase
        .from('enrollments')
        .delete()
        .eq('student_id', userId)
        .eq('course_id', courseId);
  }

  // 获取课程详情
  Future<Course> getCourseDetail(String courseId) async {
    final response = await _supabase
        .from('courses')
        .select('''
          *,
          profiles:instructor_id(full_name, avatar_url, bio),
          course_modules(
            *,
            course_lessons(*)
          ),
          enrollments(count)
        ''')
        .eq('id', courseId)
        .single();

    return Course.fromJson(response);
  }

  // 检查是否已注册
  Future<bool> isEnrolled(String courseId) async {
    final userId = _supabase.auth.currentUser?.id;
    if (userId == null) return false;

    final response = await _supabase
        .from('enrollments')
        .select()
        .eq('student_id', userId)
        .eq('course_id', courseId)
        .maybeSingle();

    return response != null;
  }
}
```

### 步骤18：课程列表界面

```dart
// lib/features/courses/presentation/pages/courses_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:cached_network_image/cached_network_image.dart';
import '../../../models/course_model.dart';
import '../providers/course_providers.dart';

class CoursesPage extends ConsumerStatefulWidget {
  const CoursesPage({super.key});

  @override
  ConsumerState<CoursesPage> createState() => _CoursesPageState();
}

class _CoursesPageState extends ConsumerState<CoursesPage>
    with SingleTickerProviderStateMixin {
  late TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: 2, vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('课程'),
        bottom: TabBar(
          controller: _tabController,
          tabs: const [
            Tab(text: '发现课程'),
            Tab(text: '我的课程'),
          ],
        ),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            onPressed: () => _showSearch(),
          ),
        ],
      ),
      body: TabBarView(
        controller: _tabController,
        children: [
          _buildDiscoverTab(),
          _buildMyCoursesTab(),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _createCourse(),
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildDiscoverTab() {
    final coursesAsync = ref.watch(publishedCoursesProvider);

    return coursesAsync.when(
      data: (courses) => RefreshIndicator(
        onRefresh: () => ref.refresh(publishedCoursesProvider.future),
        child: ListView.builder(
          padding: const EdgeInsets.all(16),
          itemCount: courses.length,
          itemBuilder: (context, index) {
            return _buildCourseCard(courses[index]);
          },
        ),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('加载失败: $error'),
            ElevatedButton(
              onPressed: () => ref.refresh(publishedCoursesProvider),
              child: const Text('重试'),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildMyCoursesTab() {
    final coursesAsync = ref.watch(enrolledCoursesProvider);

    return coursesAsync.when(
      data: (courses) => courses.isEmpty
          ? const Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(Icons.school_outlined, size: 64, color: Colors.grey),
                  SizedBox(height: 16),
                  Text('还没有注册任何课程'),
                  Text('去发现页面找找感兴趣的课程吧！'),
                ],
              ),
            )
          : RefreshIndicator(
              onRefresh: () => ref.refresh(enrolledCoursesProvider.future),
              child: ListView.builder(
                padding: const EdgeInsets.all(16),
                itemCount: courses.length,
                itemBuilder: (context, index) {
                  return _buildCourseCard(courses[index], showProgress: true);
                },
              ),
            ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(
        child: Text('加载失败: $error'),
      ),
    );
  }

  Widget _buildCourseCard(Course course, {bool showProgress = false}) {
    return Card(
      margin: const EdgeInsets.only(bottom: 16),
      child: InkWell(
        onTap: () => _navigateToCourseDetail(course.id),
        borderRadius: BorderRadius.circular(12),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // 课程缩略图
            ClipRRect(
              borderRadius: const BorderRadius.vertical(top: Radius.circular(12)),
              child: AspectRatio(
                aspectRatio: 16 / 9,
                child: course.thumbnailUrl != null
                    ? CachedNetworkImage(
                        imageUrl: course.thumbnailUrl!,
                        fit: BoxFit.cover,
                        placeholder: (context, url) => Container(
                          color: Colors.grey[300],
                          child: const Center(child: CircularProgressIndicator()),
                        ),
                        errorWidget: (context, url, error) => Container(
                          color: Colors.grey[300],
                          child: const Icon(Icons.error),
                        ),
                      )
                    : Container(
                        color: Colors.grey[300],
                        child: const Icon(Icons.play_circle_outline, size: 48),
                      ),
              ),
            ),

            Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // 课程标题
                  Text(
                    course.title,
                    style: const TextStyle(
                      fontSize: 18,
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),

                  if (course.description != null) ...[
                    const SizedBox(height: 8),
                    Text(
                      course.description!,
                      style: TextStyle(
                        color: Colors.grey[600],
                        fontSize: 14,
                      ),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],

                  const SizedBox(height: 12),

                  // 课程信息行
                  Row(
                    children: [
                      // 价格
                      if (course.price > 0)
                        Container(
                          padding: const EdgeInsets.symmetric(
                            horizontal: 8,
                            vertical: 4,
                          ),
                          decoration: BoxDecoration(
                            color: Colors.green[100],
                            borderRadius: BorderRadius.circular(4),
                          ),
                          child: Text(
                            '¥${course.price.toStringAsFixed(0)}',
                            style: TextStyle(
                              color: Colors.green[700],
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        )
                      else
                        Container(
                          padding: const EdgeInsets.symmetric(
                            horizontal: 8,
                            vertical: 4,
                          ),
                          decoration: BoxDecoration(
                            color: Colors.blue[100],
                            borderRadius: BorderRadius.circular(4),
                          ),
                          child: Text(
                            '免费',
                            style: TextStyle(
                              color: Colors.blue[700],
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        ),

                      const SizedBox(width: 8),

                      // 课时
                      if (course.durationHours > 0) ...[
                        Icon(Icons.access_time, size: 16, color: Colors.grey[600]),
                        const SizedBox(width: 4),
                        Text(
                          '${course.durationHours}小时',
                          style: TextStyle(color: Colors.grey[600]),
                        ),
                      ],

                      const Spacer(),

                      // 注册/学习按钮
                      _buildActionButton(course),
                    ],
                  ),

                  // 学习进度（如果已注册）
                  if (showProgress) ...[
                    const SizedBox(height: 12),
                    LinearProgressIndicator(
                      value: 0.3, // TODO: 从实际数据获取进度
                      backgroundColor: Colors.grey[300],
                    ),
                    const SizedBox(height: 4),
                    Text(
                      '已完成 30%',
                      style: TextStyle(
                        fontSize: 12,
                        color: Colors.grey[600],
                      ),
                    ),
                  ],
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildActionButton(Course course) {
    final isEnrolledAsync = ref.watch(isEnrolledProvider(course.id));

    return isEnrolledAsync.when(
      data: (isEnrolled) => ElevatedButton(
        onPressed: isEnrolled
            ? () => _navigateToCourseDetail(course.id)
            : () => _enrollCourse(course),
        style: ElevatedButton.styleFrom(
          backgroundColor: isEnrolled ? Colors.green : Colors.blue,
          foregroundColor: Colors.white,
        ),
        child: Text(isEnrolled ? '继续学习' : '立即注册'),
      ),
      loading: () => const SizedBox(
        width: 80,
        height: 36,
        child: Center(child: CircularProgressIndicator()),
      ),
      error: (_, __) => const SizedBox.shrink(),
    );
  }

  void _navigateToCourseDetail(String courseId) {
    Navigator.pushNamed(context, '/courses/$courseId');
  }

  void _enrollCourse(Course course) async {
    try {
      await ref.read(courseServiceProvider).enrollCourse(course.id);
      ref.invalidate(isEnrolledProvider(course.id));
      ref.invalidate(enrolledCoursesProvider);
      
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('成功注册课程：${course.title}')),
        );
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('注册失败: $e')),
        );
      }
    }
  }

  void _createCourse() {
    Navigator.pushNamed(context, '/courses/create');
  }

  void _showSearch() {
    showSearch(
      context: context,
      delegate: CourseSearchDelegate(ref),
    );
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }
}
```

---

## 📱 第九阶段：测试与部署 (第26-30天)

### 步骤19：测试实现

```bash
# 开始测试阶段
/pm:issue-start [测试任务ID]
```

#### 19.1 单元测试
```dart
// test/services/auth_service_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:canvas_learning_platform/shared/services/supabase_service.dart';

void main() {
  group('AuthService Tests', () {
    late MockSupabaseClient mockClient;
    late AuthService authService;

    setUp(() {
      mockClient = MockSupabaseClient();
      authService = AuthService();
    });

    test('should sign in with valid credentials', () async {
      // Arrange
      const email = 'test@example.com';
      const password = 'password123';
      
      // Act
      final result = await authService.signInWithEmail(email, password);
      
      // Assert
      expect(result.user, isNotNull);
    });

    test('should throw exception with invalid credentials', () async {
      // Arrange
      const email = 'invalid@example.com';
      const password = 'wrongpassword';
      
      // Act & Assert
      expect(
        () => authService.signInWithEmail(email, password),
        throwsA(isA<AuthException>()),
      );
    });
  });
}
```

#### 19.2 Widget测试
```dart
// test/widgets/course_card_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:canvas_learning_platform/features/courses/presentation/widgets/course_card.dart';
import 'package:canvas_learning_platform/models/course_model.dart';

void main() {
  group('CourseCard Widget Tests', () {
    late Course testCourse;

    setUp(() {
      testCourse = Course(
        id: '1',
        title: 'Test Course',
        description: 'Test Description',
        instructorId: '1',
        status: CourseStatus.published,
        price: 99.0,
        durationHours: 10,
        createdAt: DateTime.now(),
        updatedAt: DateTime.now(),
      );
    });

    testWidgets('should display course title and description', (tester) async {
      // Arrange & Act
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CourseCard(course: testCourse),
          ),
        ),
      );

      // Assert
      expect(find.text('Test Course'), findsOneWidget);
      expect(find.text('Test Description'), findsOneWidget);
      expect(find.text('¥99'), findsOneWidget);
    });

    testWidgets('should show free badge for free courses', (tester) async {
      // Arrange
      final freeCourse = testCourse.copyWith(price: 0);
      
      // Act
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CourseCard(course: freeCourse),
          ),
        ),
      );

      // Assert
      expect(find.text('免费'), findsOneWidget);
    });
  });
}
```

#### 19.3 集成测试
```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:canvas_learning_platform/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('App Integration Tests', () {
    testWidgets('complete user flow test', (tester) async {
      // 启动应用
      app.main();
      await tester.pumpAndSettle();

      // 验证启动页面
      expect(find.byType(SplashPage), findsOneWidget);
      await tester.pumpAndSettle(const Duration(seconds: 3));

      // 导航到登录页面
      expect(find.byType(LoginPage), findsOneWidget);

      // 输入登录信息
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );

      // 点击登录按钮
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle();

      // 验证登录成功，导航到仪表板
      expect(find.byType(DashboardPage), findsOneWidget);

      // 导航到课程页面
      await tester.tap(find.byIcon(Icons.school));
      await tester.pumpAndSettle();

      expect(find.byType(CoursesPage), findsOneWidget);
    });
  });
}
```

### 步骤20：性能优化

#### 20.1 代码分割和懒加载
```dart
// lib/router/app_router.dart
final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    routes: [
      // 懒加载路由
      GoRoute(
        path: '/courses',
        builder: (context, state) => const CoursesPage(),
        routes: [
          GoRoute(
            path: '/:id',
            builder: (context, state) {
              return FutureBuilder(
                future: _loadCourseDetailPage(),
                builder: (context, snapshot) {
                  if (snapshot.hasData) {
                    return snapshot.data!;
                  }
                  return const CircularProgressIndicator();
                },
              );
            },
          ),
        ],
      ),
    ],
  );
});

Future<Widget> _loadCourseDetailPage() async {
  // 懒加载课程详情页面
  final module = await import('package:canvas_learning_platform/features/courses/presentation/pages/course_detail_page.dart');
  return module.CourseDetailPage();
}
```

#### 20.2 图像缓存优化
```dart
// lib/shared/widgets/optimized_image.dart
class OptimizedImage extends StatelessWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  final BoxFit fit;

  const OptimizedImage({
    super.key,
    required this.imageUrl,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
  });

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: imageUrl,
      width: width,
      height: height,
      fit: fit,
      memCacheWidth: width?.toInt(),
      memCacheHeight: height?.toInt(),
      placeholder: (context, url) => Container(
        width: width,
        height: height,
        color: Colors.grey[300],
        child: const Center(
          child: CircularProgressIndicator(strokeWidth: 2),
        ),
      ),
      errorWidget: (context, url, error) => Container(
        width: width,
        height: height,
        color: Colors.grey[300],
        child: const Icon(Icons.error),
      ),
    );
  }
}
```

### 步骤21：部署配置

#### 21.1 环境配置
```dart
// lib/core/config/app_config.dart
class AppConfig {
  static const String _supabaseUrlKey = 'SUPABASE_URL';
  static const String _supabaseAnonKeyKey = 'SUPABASE_ANON_KEY';
  static const String _liveKitUrlKey = 'LIVEKIT_URL';

  static String get supabaseUrl => 
      const String.fromEnvironment(_supabaseUrlKey);
  
  static String get supabaseAnonKey => 
      const String.fromEnvironment(_supabaseAnonKeyKey);
  
  static String get liveKitUrl => 
      const String.fromEnvironment(_liveKitUrlKey);

  static bool get isProduction => 
      const bool.fromEnvironment('dart.vm.product');
}
```

#### 21.2 Docker配置
```dockerfile
# Dockerfile
FROM cirrusci/flutter:stable

WORKDIR /app
COPY . .

RUN flutter pub get
RUN flutter build web --release

FROM nginx:alpine
COPY --from=0 /app/build/web /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

#### 21.3 部署脚本
```bash
#!/bin/bash
# deploy.sh

# 构建Web版本
flutter build web --release --dart-define=SUPABASE_URL=$SUPABASE_URL --dart-define=SUPABASE_ANON_KEY=$SUPABASE_ANON_KEY

# 部署到服务器
rsync -avz --delete build/web/ user@server:/var/www/canvas-platform/

# 构建移动应用
flutter build apk --release
flutter build ios --release

echo "部署完成!"
```

### 步骤22：运行测试和构建
```bash
# 运行所有测试
flutter test

# 运行集成测试
flutter drive --target=test_driver/app.dart

# 构建生产版本
flutter build web --release
flutter build apk --release
flutter build ios --release

# 同步项目进度
/pm:issue-sync [测试任务ID]
/pm:epic-show canvas-learning-platform
```

---

## 📊 项目管理和跟踪

### CCPM命令使用指南

在整个开发过程中，使用以下命令跟踪项目进度：

```bash
# 查看总体项目状态
/pm:status

# 获取下一个优先任务  
/pm:next

# 开始特定任务
/pm:issue-start [GitHub Issue ID]

# 同步任务进度到GitHub
/pm:issue-sync [GitHub Issue ID]

# 查看Epic状态
/pm:epic-show canvas-learning-platform

# 每日站会报告
/pm:standup

# 查看被阻塞的任务
/pm:blocked

# 搜索项目内容
/pm:search "关键词"
```

### 关键里程碑检查点

- **第1周末**: ✅ 项目初始化，环境搭建完成
- **第2周末**: ✅ 数据库设计，认证系统完成
- **第3周末**: ✅ 核心UI界面开发完成
- **第4周末**: ✅ 实时功能(LiveKit + 聊天)集成完成
- **第5周末**: ✅ 课程管理系统完成
- **第6周末**: ✅ 测试通过，生产环境部署完成

---

## 🎯 成功标准

### 技术指标
- [ ] 应用启动时间 < 3秒
- [ ] 页面切换响应时间 < 500ms  
- [ ] 视频通话延迟 < 100ms
- [ ] 聊天消息实时同步 < 200ms
- [ ] 支持1000+并发用户
- [ ] 99.9%可用性

### 功能完整性
- [ ] 用户认证和权限管理
- [ ] 多角色支持(学生/教师/管理员)
- [ ] 课程创建和管理
- [ ] 实时视频教学
- [ ] 即时聊天系统  
- [ ] 作业提交和批改
- [ ] 学习进度跟踪
- [ ] 移动端适配

### 用户体验
- [ ] 直观的用户界面
- [ ] 响应式设计
- [ ] 离线功能支持
- [ ] 多语言支持(可选)
- [ ] 无障碍功能支持

---

## 🔧 故障排除

### 常见问题解决方案

#### 1. Supabase连接问题
```bash
# 检查环境变量
echo $SUPABASE_URL
echo $SUPABASE_ANON_KEY

# 重新启动本地Supabase
supabase stop
supabase start
```

#### 2. LiveKit连接失败
```bash
# 检查权限设置
flutter doctor
# 确保摄像头和麦克风权限已授权
```

#### 3. Flutter构建错误
```bash
# 清理构建缓存
flutter clean
flutter pub get
flutter pub run build_runner build --delete-conflicting-outputs
```

#### 4. 数据库迁移问题
```bash
# 重置本地数据库
supabase db reset
supabase db push
```

---

## 📚 参考资源

### 官方文档
- [Flutter文档](https://flutter.dev/docs)
- [Supabase文档](https://supabase.com/docs)
- [LiveKit文档](https://docs.livekit.io)
- [CCPM使用指南](https://github.com/automazeio/ccpm)

### 社区资源
- [Flutter中文网](https://flutter.cn)
- [Supabase中文社区](https://supabase.com/docs/guides/getting-started)
- [LiveKit示例项目](https://github.com/livekit/livekit-examples)

### 学习资料
- Flutter实战课程
- Supabase全栈开发教程  
- WebRTC音视频开发指南
- 教育平台设计最佳实践

---

**祝您开发顺利！🚀**

使用此指南，您将能够构建一个功能完整、性能优秀的现代化Canvas学习平台。记住要频繁使用CCPM命令跟踪进度，确保项目按计划推进。