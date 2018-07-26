# 1. 引入依赖

```text
npm i -S @ngrx/core
npm i -S @ngrx/effects
npm i -S @ngrx/router-store
npm i -S @ngrx/store
npm i -S @ngrx/store-devtools
```

# 2. @ngrx/store的用法

## 2.1 创建Action

```typescript
import { Action } from '@ngrx/store';

export enum UserActionEnum {
  LOGIN = 'login',
  LOGIN_SUCCESS = 'login_success',
  LOGIN_FAIL = 'login_fail'
}

export class BaseAction implements Action {
  constructor(
    public type: string,
    public payload?: any
  ) {}
}

export class LoginSuccessAction extends BaseAction {
  constructor(){
    super(UserActionEnum.LOGIN_SUCCESS)
  }
}

export class LoginFailAction extends BaseAction {
  constructor(){
    super(UserActionEnum.LOGIN_FAIL)
  }
}

export class LoginAction extends BaseAction {
  constructor(userID: string, password: string) {
    super(UserActionEnum.LOGIN, {
      userID: userID,
      password: password
    })
  }
}
```

## 2.2 创建Reducer

```typescript
import {UserActionEnum, BaseAction} from '../actions/user.action';
import { Action } from '@ngrx/store';

export interface UserState {
  userId: string;
  password: string;
  loginStatus: boolean;
}

const initialState: UserState = {
  userId: '',
  password: '',
  loginStatus: false,
};

export function userReducer(state = initialState, action: BaseAction): UserState {
  const newState = {...state};
  switch (action.type) {
    case UserActionEnum.LOGIN_SUCCESS:
      newState.loginStatus = true;
      break;
    case UserActionEnum.LOGIN_FAIL:
      newState.loginStatus = false;
      break;

    default:
      break;
  }
  return newState;
}
```

## 2.3 在AppModule添加import

```typescript
import { userReducer } from './redurces/user.reducer';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { StoreModule } from '@ngrx/store';

@NgModule({
  declarations: [ /*省略*/ ],
  imports: [
    StoreModule.forRoot({
      user: userReducer
    }),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // 只显示最近25个状态
      logOnly: environment.production, // 只有在production才输出记录
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## 2.4 在Component中使用redux
```typescript
import { Component, OnInit } from '@angular/core';
import { Store, select } from '@ngrx/store';
import { UserState } from './redurces/user.reducer';
import { Observable } from 'rxjs';
import { LoginSuccessAction, LoginFailAction, LoginAction } from './actions/user.action';
import { FormBuilder, FormGroup } from '@angular/forms';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit {

  title = 'app';
  loginStatus$: Observable<boolean>;
  loginForm: FormGroup;
  constructor(
    private store: Store<UserState>,
    private fb: FormBuilder
  ) {
    this.loginStatus$ = store.select('user', 'loginStatus');
    this.loginStatus$.subscribe(loginStatus => {
      console.log(loginStatus);
    })
  }

  ngOnInit(): void {
    this.loginForm = this.fb.group({
      userID: this.fb.control(''),
      password: this.fb.control('')
    });
  }

  loginSuccess() {
    this.store.dispatch(new LoginSuccessAction());
  }

  loginFail() {
    this.store.dispatch(new LoginFailAction());
  }

  onSubmit(): void {
    const formValue = this.loginForm.value;
    this.store.dispatch(new LoginAction(formValue.userID, formValue.password));
  }
}
```

HTML:
```html
<p>
  {{loginStatus$ | async}}
</p>
```


# 3. @ngrx/effects的用法

## 3.1 创建UserEffects
```typescript
import { Actions, Effect, ofType } from "@ngrx/effects";
import { Injectable } from "@angular/core";
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { UserActionEnum, LoginSuccessAction, LoginFailAction, LoginAction, BaseAction } from '../actions/user.action';

@Injectable()
export class UserEffects{

  @Effect()
  user$: Observable<BaseAction> = this.actions$
    .pipe(
      ofType<LoginAction>(UserActionEnum.LOGIN),  // 监听LoginAction，当监听到的执行下面的操作
      map((action, index) => {
        // 验证用户名和密码，然后返回不同的Action
        if (action.payload.userID === 'hushukang' && action.payload.password === '12345') {
          return new LoginSuccessAction();
        } else {
          return new LoginFailAction();
        }
      })
    );

  constructor(private actions$: Actions) { }
}
```

## 3.2 在AppModule添加import

```typescript
import { userReducer } from './redurces/user.reducer';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { StoreModule } from '@ngrx/store';
import { UserEffects } from './effects/user.effect';
import { EffectsModule } from '@ngrx/effects';

@NgModule({
  declarations: [ /*省略*/ ],
  imports: [
    StoreModule.forRoot({
      user: userReducer
    }),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // 只显示最近25个状态
      logOnly: environment.production, // 只有在production才输出记录
    }),
    EffectsModule.forRoot([
      UserEffects
    ])
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```