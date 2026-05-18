# OAuth Login — Google & Facebook

## Mục tiêu

Cho phép user đăng nhập bằng tài khoản Google (Gmail) hoặc Facebook mà không cần nhập mật khẩu. Sau khi OAuth thành công, backend trả về JWT giống hệt luồng login thông thường.

---

## Quyết định thiết kế

### 1. Merge account theo email

Nếu email từ OAuth đã tồn tại trong DB (đăng ký bằng email/password trước), hệ thống **tự động liên kết** thay vì báo lỗi. User đăng nhập được bằng cả hai cách.

### 2. Trả token về frontend

Callback từ Google/Facebook là redirect HTTP, không phải JSON response. Cách xử lý:

```
GET /api/auth/google/callback
  → OAuthLoginUseCase trả về JWT
  → Redirect: {FRONTEND_URL}/auth/callback?token=<jwt>
  → Frontend đọc query param, lưu vào localStorage
```

### 3. User tạo qua OAuth không có password

`passwordHash` sẽ là chuỗi rỗng `""`. User không thể đăng nhập bằng email/password trừ khi họ tự set password sau.

### 4. Username tự động sinh

Lấy từ `profile.displayName` (Google) hoặc `profile.displayName` (Facebook), nếu trùng thì thêm suffix ngẫu nhiên (`_abc123`).

---

## Env vars cần thêm

```env
# Google
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_CALLBACK_URL=http://localhost:4000/api/auth/google/callback

# Facebook
FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET=
FACEBOOK_CALLBACK_URL=http://localhost:4000/api/auth/facebook/callback

# Frontend redirect sau OAuth
FRONTEND_URL=http://localhost:3000
```

---

## Schema thay đổi

**File:** `src/infrastructure/databases/schemas/user.schema.ts`

Thêm 3 field vào UserSchema:

```ts
@Prop({ required: false })
googleId?: string;

@Prop({ required: false })
facebookId?: string;

@Prop({ enum: ['local', 'google', 'facebook'], default: 'local' })
provider: string;
```

`passwordHash` bỏ `required: true` → `required: false` (vì OAuth user không có password).

**File:** `src/core/domain/entities/user.entity.ts`

Thêm field tương ứng:

```ts
googleId?: string;
facebookId?: string;
provider?: string;
```

---

## Các file cần tạo mới

### 1. Strategies

**`src/infrastructure/auth/strategies/google.strategy.ts`**

```ts
@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor(configService: ConfigService) {
    super({
      clientID: configService.get('GOOGLE_CLIENT_ID'),
      clientSecret: configService.get('GOOGLE_CLIENT_SECRET'),
      callbackURL: configService.get('GOOGLE_CALLBACK_URL'),
      scope: ['email', 'profile'],
    });
  }

  async validate(accessToken, refreshToken, profile, done) {
    // trả về OAuthProfile để controller dùng
    done(null, {
      provider: 'google',
      providerId: profile.id,
      email: profile.emails[0].value,
      fullName: profile.displayName,
      profilePic: profile.photos?.[0]?.value,
    });
  }
}
```

**`src/infrastructure/auth/strategies/facebook.strategy.ts`**

Tương tự, dùng `passport-facebook`, thêm `profileFields: ['id', 'emails', 'name', 'picture']`.

---

### 2. Use-case

**`src/use-case/auth/oauth-login.use-case.ts`**

Logic:
1. Tìm user theo `googleId`/`facebookId` (tùy provider)
2. Nếu không tìm thấy → tìm theo email
3. Nếu vẫn không tìm thấy → tạo user mới
4. Nếu tìm thấy qua email nhưng chưa có `providerId` → gắn `providerId` vào user (liên kết account)
5. Gọi `JwtService.sign()` → trả về `access_token`

```ts
async execute(profile: OAuthProfile): Promise<{ access_token: string }> {
  // ... find-or-create logic
}
```

**Interface `OAuthProfile`** (đặt ở `src/core/domain/interfaces/oauth-profile.interface.ts`):

```ts
export interface OAuthProfile {
  provider: 'google' | 'facebook';
  providerId: string;
  email: string;
  fullName: string;
  profilePic?: string;
}
```

---

### 3. Repository thay đổi

**`src/core/domain/repositories/user.repository.interface.ts`**

Thêm 2 method:

```ts
findByGoogleId(googleId: string): Promise<User | null>;
findByFacebookId(facebookId: string): Promise<User | null>;
```

**`src/infrastructure/databases/repositories/user.repository.ts`**

Implement 2 method trên.

---

## Các file cần sửa

### `src/presentation/controllers/auth.controller.ts`

Thêm 4 route:

```ts
// Google
@Get('google')
@UseGuards(AuthGuard('google'))
googleAuth() {}

@Get('google/callback')
@UseGuards(AuthGuard('google'))
async googleCallback(@Request() req, @Res() res) {
  const { access_token } = await this.oauthLoginUseCase.execute(req.user);
  res.redirect(`${frontendUrl}/auth/callback?token=${access_token}`);
}

// Facebook
@Get('facebook')
@UseGuards(AuthGuard('facebook'))
facebookAuth() {}

@Get('facebook/callback')
@UseGuards(AuthGuard('facebook'))
async facebookCallback(@Request() req, @Res() res) {
  const { access_token } = await this.oauthLoginUseCase.execute(req.user);
  res.redirect(`${frontendUrl}/auth/callback?token=${access_token}`);
}
```

### `src/infrastructure/auth/auth.module.ts`

- Import `GoogleStrategy`, `FacebookStrategy`
- Thêm `OAuthLoginUseCase` vào providers

### `src/infrastructure/config/` (env validation)

Thêm các key mới vào schema validation nếu có (optional, vì OAuth là tính năng có thể tắt).

---

## Thứ tự implement

```
1. Cài packages
   npm i passport-google-oauth20 passport-facebook
   npm i -D @types/passport-google-oauth20 @types/passport-facebook

2. Thêm env vars vào .env

3. Cập nhật User entity + UserSchema (thêm googleId, facebookId, provider)

4. Thêm findByGoogleId / findByFacebookId vào repo interface + implementation

5. Tạo OAuthProfile interface

6. Tạo OAuthLoginUseCase

7. Tạo GoogleStrategy

8. Tạo FacebookStrategy

9. Sửa AuthController (thêm 4 route)

10. Sửa AuthModule (đăng ký strategy + use-case mới)

11. Test thủ công qua browser
```

---

## Lưu ý

- **Facebook yêu cầu HTTPS** cho callback URL khi đăng ký app production. Local dev có thể dùng `http://` nếu set app mode = Development trên Facebook Developer Console.
- **Google cần verify app** nếu muốn dùng production. Dev mode chỉ cho phép test users đã được thêm vào danh sách.
- `gender` là required trong schema hiện tại. User tạo qua OAuth sẽ không có gender → cần bỏ `required: true` hoặc set default `'other'`.
