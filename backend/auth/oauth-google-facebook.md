# OAuth Login — Google & Facebook

## Goal

Allow users to sign in with their Google (Gmail) or Facebook account without entering a password. After a successful OAuth flow, the backend returns a JWT identical to the standard login flow.

---

## Design Decisions

### 1. Account Merging by Email

If the OAuth email already exists in the DB (registered via email/password before), the system **automatically links** the accounts instead of throwing an error. The user can then log in with either method.

### 2. Returning the Token to the Frontend

The Google/Facebook callback is an HTTP redirect, not a JSON response. Handling:

```
GET /api/auth/google/callback
  → OAuthLoginUseCase returns JWT
  → Redirect: {FRONTEND_URL}/auth/callback?token=<jwt>
  → Frontend reads query param, stores in localStorage
```

### 3. OAuth Users Have No Password

`passwordHash` will be an empty string `""`. The user cannot log in via email/password unless they manually set a password later.

### 4. Auto-generated Username

Taken from `profile.displayName` (Google or Facebook). If it conflicts with an existing username, a random suffix is appended (`_abc123`).

---

## Required Env Vars

```env
# Google
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_CALLBACK_URL=http://localhost:4000/api/auth/google/callback

# Facebook
FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET=
FACEBOOK_CALLBACK_URL=http://localhost:4000/api/auth/facebook/callback

# Frontend redirect after OAuth
FRONTEND_URL=http://localhost:3000
```

---

## Schema Changes

**File:** `src/infrastructure/databases/schemas/user.schema.ts`

Add 3 fields to UserSchema:

```ts
@Prop({ required: false })
googleId?: string;

@Prop({ required: false })
facebookId?: string;

@Prop({ enum: ['local', 'google', 'facebook'], default: 'local' })
provider: string;
```

`passwordHash` — change `required: true` → `required: false` (OAuth users have no password).

**File:** `src/core/domain/entities/user.entity.ts`

Add corresponding fields:

```ts
googleId?: string;
facebookId?: string;
provider?: string;
```

---

## New Files to Create

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
    // return OAuthProfile for the controller to use
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

Same pattern using `passport-facebook`, add `profileFields: ['id', 'emails', 'name', 'picture']`.

---

### 2. Use Case

**`src/use-case/auth/oauth-login.use-case.ts`**

Logic:
1. Find user by `googleId` / `facebookId` (depending on provider)
2. If not found → find by email
3. If still not found → create new user
4. If found by email but missing `providerId` → attach `providerId` to user (link account)
5. Call `JwtService.sign()` → return `access_token`

```ts
async execute(profile: OAuthProfile): Promise<{ access_token: string }> {
  // ... find-or-create logic
}
```

**`OAuthProfile` interface** (place at `src/core/domain/interfaces/oauth-profile.interface.ts`):

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

### 3. Repository Changes

**`src/core/domain/repositories/user.repository.interface.ts`**

Add 2 methods:

```ts
findByGoogleId(googleId: string): Promise<User | null>;
findByFacebookId(facebookId: string): Promise<User | null>;
```

**`src/infrastructure/databases/repositories/user.repository.ts`**

Implement both methods above.

---

## Files to Modify

### `src/presentation/controllers/auth.controller.ts`

Add 4 routes:

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
- Add `OAuthLoginUseCase` to providers

### `src/infrastructure/config/` (env validation)

Add the new keys to the validation schema if applicable (optional — OAuth can be treated as a toggleable feature).

---

## Implementation Order

```
1. Install packages
   npm i passport-google-oauth20 passport-facebook
   npm i -D @types/passport-google-oauth20 @types/passport-facebook

2. Add env vars to .env

3. Update User entity + UserSchema (add googleId, facebookId, provider)

4. Add findByGoogleId / findByFacebookId to repo interface + implementation

5. Create OAuthProfile interface

6. Create OAuthLoginUseCase

7. Create GoogleStrategy

8. Create FacebookStrategy

9. Update AuthController (add 4 routes)

10. Update AuthModule (register new strategies + use case)

11. Manual test via browser
```

---

## Notes

- **Facebook requires HTTPS** for the callback URL in production. For local dev, `http://` works if the app mode is set to Development in the Facebook Developer Console.
- **Google requires app verification** for production use. Dev mode only allows test users explicitly added to the allowlist.
- `gender` is currently required in the schema. OAuth users won't have a gender value — either remove `required: true` or set a default of `'other'`.
