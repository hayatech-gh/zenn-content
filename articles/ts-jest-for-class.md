---
title: 'Jestを使ったクラスをテスト手法まとめ'
emoji: '☕️'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['Jest', 'TypeScript', 'JavaScript']
published: true
date: '2025.08.16'
---

# Jest を使ったクラスをテスト手法まとめ

業務で Jest を使う機会が多くなってきたので、おさらいとしてクラスを対象としたテスト手法をまとめてみました。
クラスを使ったコード例、マッチャー、モック機能について、紹介します。

---

## 1\. Jest の基本的なテストコード

```ts
// src/UserService.ts
import { AuthApi } from './AuthApi';

export class UserService {
  constructor(private authApi: AuthApi) {}

  async login(username: string, password: string): Promise<boolean> {
    const token = await this.authApi.authenticate(username, password);
    return token !== null;
  }

  async isAdmin(username: string): Promise<boolean> {
    const roles = await this.authApi.getUserRoles(username);
    return roles.includes('admin');
  }
}
```

```ts
// src/AuthApi.ts
export class AuthApi {
  async authenticate(username: string, password: string): Promise<string | null> {
    // 外部API呼び出しを想定
    if (username === 'admin' && password === 'secret') return 'token123';
    return null;
  }

  async getUserRoles(username: string): Promise<string[]> {
    if (username === 'admin') return ['user', 'admin'];
    return ['user'];
  }
}
```

コード例として挙げている、`UserService`は、ユーザー認証や権限管理を処理するクラスです。

- `login` メソッドは、ユーザー名とパスワードを受け取り、`AuthApi`を通じて認証トークンを取得します。
- トークンが取得できれば `true` を返し、取得できなければ `false` を返します。
- `isAdmin` メソッドは、ユーザー名から権限情報を取得し、管理者権限（`admin`）を持っているかどうかを判定します。

---

## 基本的なテストコード

```ts
// src/__tests__/UserService.test.ts
import { UserService } from '../UserService';
import { AuthApi } from '../AuthApi';

describe('UserService', () => {
  let authApi: AuthApi;
  let userService: UserService;

  beforeEach(() => {
    authApi = new AuthApi();
    userService = new UserService(authApi);
  });

  test('loginは正しい認証情報でtrueを返す', async () => {
    await expect(userService.login('admin', 'secret')).resolves.toBe(true);
  });

  test('loginは間違った認証情報でfalseを返す', async () => {
    await expect(userService.login('admin', 'wrong')).resolves.toBe(false);
  });

  test('isAdminはadminユーザーでtrueを返す', async () => {
    await expect(userService.isAdmin('admin')).resolves.toBe(true);
  });

  test('isAdminは一般ユーザーでfalseを返す', async () => {
    await expect(userService.isAdmin('user')).resolves.toBe(false);
  });
});
```

テストでは、以下の点に注目しています。

- `beforeEach` を使って、テストごとに初期化し、新しい `UserService` インスタンスを生成します。
- `login` メソッドや `isAdmin` メソッドは非同期関数なので、`await expect(...).resolves.toBe(...)` の形で Promise の結果を検証しています。

---

## マッチャーの種類

JEST でよく使われるマッチャーの種類を表にまとめました。

| 種類           | 説明                                 | 例                                           |
| -------------- | ------------------------------------ | -------------------------------------------- |
| 値の一致       | 値が完全に一致するか確認             | `toBe(5)`                                    |
| 値の内容       | オブジェクトや配列の内容が一致するか | `toEqual({a:1})`                             |
| 例外           | 関数が例外を投げるか                 | `toThrow("error")`                           |
| 真偽値         | true/false を確認                    | `toBeTruthy() / toBeFalsy()`                 |
| null/undefined | null や undefined を確認             | `toBeNull() / toBeUndefined()`               |
| 配列/文字列    | 配列や文字列に要素が含まれるか       | `toContain("value")`                         |
| 非同期         | Promise の結果を検証                 | `resolves.toBe(value)` / `rejects.toThrow()` |

---

## マッチャーの例

上記の種類表を参考に、UserService クラスのテストでいくつかのマッチャーを試します。

```ts
test("getUserRolesの結果に'user'が含まれる", async () => {
  const roles = await authApi.getUserRoles('admin');
  expect(roles).toContain('user'); // 配列/文字列
});

test('authenticateはnullでない値を返す場合がある', async () => {
  const token = await authApi.authenticate('admin', 'secret');
  expect(token).not.toBeNull(); // null/undefined
});

test('loginのPromiseはtrueを返す', async () => {
  await expect(userService.login('admin', 'secret')).resolves.toBe(true); // 非同期
});
```

---

## モックの使い方

外部 API 呼び出しをモックしてテストします。

```ts
// src/__tests__/UserService.mock.test.ts
import { UserService } from '../UserService';
import { AuthApi } from '../AuthApi';

jest.mock('../AuthApi'); // AuthApiをモック

describe('UserService with mocked AuthApi', () => {
  let authApiMock: jest.Mocked<AuthApi>;
  let userService: UserService;

  beforeEach(() => {
    authApiMock = new AuthApi() as jest.Mocked<AuthApi>;
    userService = new UserService(authApiMock);
  });

  test('loginはモックでtrueを返す', async () => {
    authApiMock.authenticate.mockResolvedValue('fakeToken');
    await expect(userService.login('anyuser', 'anypass')).resolves.toBe(true);
    expect(authApiMock.authenticate).toHaveBeenCalledWith('anyuser', 'anypass');
  });

  test('isAdminはモックでfalseを返す', async () => {
    authApiMock.getUserRoles.mockResolvedValue(['user']);
    await expect(userService.isAdmin('anyuser')).resolves.toBe(false);
  });
});
```

外部 API や副作用のある処理は、テスト環境でそのまま実行すると結果が不安定になったり、テストが遅くなったりします。
そのため、`jest.mock()` を使って `AuthApi` クラスをモック化し、呼び出しごとに任意の戻り値を返すようにしています。

- `authApiMock.authenticate.mockResolvedValue("fakeToken")` によって、どんな入力でも必ず `"fakeToken"` が返るようにしています。
- `authApiMock.getUserRoles.mockResolvedValue(["user"])` によって、ユーザー権限情報を自由に設定できます。
- モックを使うことで、外部 API の挙動に依存せず、`UserService` 自体の振る舞いのみをテストできます。
- `toHaveBeenCalledWith` などのマッチャーを使うと、モック関数が期待通りの引数で呼ばれたかどうかも検証できます。
