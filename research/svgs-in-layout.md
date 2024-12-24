## Research how to use ReactComponent instead `<img />`

При переносе layout с Webpack & React17 на Vite & React18 столкнулись с тем, что иконки не хотели использоваться как ReactComponent и приходилось их использовать через `<img />`

Так было на Webpack & React17

```
import { ReactComponent as MyIcon } from ''./path/to/icon.svg'

const MyComponent = () => {
  return (
    <div>
      <MyIcon />
    </div>
  );
};

export default MyComponent;
```

При переносе компонентов с Webpack & React17 на Vite & React18, появилась ошибка, ругающаяся на ReactComponent 
![Error text svg](./images/error-text-svg.png)

Начали пробовать разные вариации, чтобы остаться на использовании ReactComponent, но в итоге, при быстром поиске, рабочим оказался только вариант ниже (первый)

1.

```  
import MyIcon from './path/to/icon.svg';

const MyComponent = () => {
  return (
    <div>
      <img src={MyIcon} /> 
    </div>
  );
};

export default MyComponent;
```

Нам конечно он не нравился и казалось, что есть решение, чтобы остаться на использовании ReactComponent

2. Также пробовали создать в корне проекта файл `photo.d.ts`

```
declare module '\*.svg' {  
 export const ReactComponent: React.FC\<React.SVGProps\<SVGSVGElement\>\>  
 const src: string  
 export default src  
}
```

И подключили его в файл `tsconfig.json`

```
 "include": \[  
   "\*\*/\*.d.ts", // или пробовали "photo.d.ts",  
 \],
```

Но не уверены, что правильно поняли, как оно работает и подключилось ли оно вообще, потому что ничего не изменилось

3. Есть также варианты избавиться от ошибки через настройку `vite.config.ts`, но там тоже не сильно получилось это сделать

Популярный способ решения это плагин `svgr`, но он почему-то тоже ничем не помог, ничего не изменил. Есть разные вариации его подключения, с доп флагами, либо не из `vite-plugin-svgr`. Самый популярный вариант приведен ниже 

``` 
import { defineConfig } from 'vite'  
import react from '@vitejs/plugin-react'  
import svgr from 'vite-plugin-svgr'

export default defineConfig({  
 plugins: \[  
   react(),  
   svgr(), // плагин  
 \],  
})
```

---

В итоге получилось найти решение, которое нас устроило, потому что мы были правы в том, что есть возможность вернуться к использованию иконок как ReactComponent

Мы попробовали добавлять приписку `?react` к импорту

```
import MyIcon from './path/to/icon.svg?react';

const MyComponent = () => {
  return (
    <div>
      <MyIcon />
    </div>
  );
};

export default MyComponent;
```

И в `vite-env.d.ts` добавить строку

```
/// <reference types="vite-plugin-svgr/client" />
```

И оно заработало!!
