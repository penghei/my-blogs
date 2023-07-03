发现了一个很不错的组件，可以控制组件渲染的优先级

> 当页面的模块有优先级之分（通常也被称为核心模块和非核心模块、高优模块和低优模块）时，进入页面后需保证高优先级模块先渲染，然后再加渲染低优模块。 该工具提供了一个较通用的解决方案，可以节省类似场景的开发成本。

属性：

- priority：数字优先级，0 最高，依次类推
- allowNextPriorityRender：是否允许下一优先级组件加载。可以手动控制某个优先级的组件是否可以渲染

使用示例：

```js
export default function PriorityDemo() {
  const [allowNext, setAllowNext] = useState(false);
  return (
    <View className="priority-root">
      <Priority priority={0}>
        <Text>测试1</Text>
      </Priority>

      <Priority priority={1} allowNextPriorityRender={allowNext}>
        <View
          onClick={() => {
            setAllowNext(true);
            // 初始化只会先渲染0再渲染1，后面的暂时不渲染
            // 这里执行完成之后，1之后的优先级都可以渲染，然后按照次序渲染
          }}
        >
          <Text>点击支持低优先级加载</Text>
        </View>
      </Priority>

      <Priority priority={2}>
        <Text>测试3</Text>
      </Priority>

      <Priority priority={3}>
        <Text>测试4</Text>
      </Priority>

      <Priority priority={3}>
        <Text>测试5</Text>
      </Priority>
    </View>
  );
}
```


代码结构如下：

- index.tsx: 组件核心代码
- PriorityComponent.tsx: 包裹了一层的组件
- Manager.ts: 核心代码，作为优先级的管理器。

代码如下：

```tsx
// index.tsx
import { createCondition as __create_condition__ } from "babel-runtime-jsx-plus";
import React from "react";
import { useState, useEffect, useRef } from "react";
import PriorityComponent from "./PriorityComponent";
import PriorityManager from "./PriorityManager";
export default function Priority(props) {
  const [show, setShow] = useState(false);
  const priorityRef = useRef(null);
  const { priority, allowNextPriorityRender = true } = props;

  const getUniKey = () => {
    if (isWeChatMiniProgram) {
      return props.__tagId;
    }

    return priorityRef.current;
  };

  useEffect(() => {
    const uniqKey = getUniKey(); // 获取组件外面包一层的view的ref，作为每个组件的key
    // 如果当前组件未渲染（默认未渲染），注册组件
    // registerComponent的回调将会在组件可以渲染时执行，效果就是让该组件渲染，同时引起之后组件（下一个优先级）的渲染
    !show &&
      !PriorityManager.isRegistered(uniqKey) &&
      PriorityManager.registerComponent(uniqKey, priority, () => {
        // 这里的callback传的是修改show，让当前组件能渲染
        // 在Manager中，通过notifyRender执行对应的callback，这里就会被执行，然后就可以渲染组件
        // 这里其实也可升级为一个发布订阅
        setShow(true);
      });

    // 如果当前组件可以渲染了，就进行渲染
    // 这一个判断主要是考虑到当前组件由其他组件引起的。
    // 比如上一个优先级的组件渲染完了，在manager中把当前优先级的组件设为approval（可渲染），那么这里会执行渲染
    // allowRenderSelf会判断当前组件是不是最高优先级的组件，如果是就会返回true，该组件就会进行渲染。初始化就从这里开始，第一个组件引起后面组件的渲染，然后依次类推
    if (!show && PriorityManager.allowRenderSelf(uniqKey)) {
      setShow(true);
    }

    return () => {
      PriorityManager.unregisterComponent(uniqKey);
    };
  });
  return (
    <View ref={priorityRef}>
      {__create_condition__([
        [
          () => show,
          () => (
            <PriorityComponent
              priority={priority}
              onGetParentRef={getUniKey}
              allowNextPriorityRender={allowNextPriorityRender}
            >
              {props.children}
            </PriorityComponent>
          ),
        ],
      ])}
    </View>
  );
}
```

```tsx
// PriorityComponent.tsx
import React from "react";
// @ts-nocheck
import { useEffect } from "react";
import PriorityManager from "./PriorityManager";
export default function PriorityComponent(props) {
  const { onGetParentRef, allowNextPriorityRender } = props;
  useEffect(() => {
    // componentDidRendered会执行传入组件的渲染，并将下一个优先级的组件触发渲染
    // 这里就是，当当前组件被渲染时，就引起下一个优先级的组件进行渲染。
    // 比如当前组件的优先级是1，那么当前组件挂载之后，就去引起优先级为2的所有组件的渲染

    // 如果某个组件的allowNextPriorityRender为false，那么连续的渲染就会被终止，
    // 直到allowNextPriorityRender改变，继续进行后续的渲染
    allowNextPriorityRender &&
      PriorityManager.componentDidRendered(onGetParentRef());
    
  });
  return <View>{props.children}</View>;
}
```

```ts
export default class Manager {
  components = new Map();
  status = new Map();
  
  callback = new Map();
  convertPriority = (priority) => {
    let p;

    if (priority === "high") {
      p = 0;
    } else if (priority === "low") {
      p = 1;
    } else {
      p = priority;
    }

    return p;
  };
  isRegistered = (component) => {
    return this.components.has(component);
  };
  registerComponent = (component, priority, callback) => {
    if (!component) {
      return;
    }

    const p = this.convertPriority(priority);

    if (typeof p !== "number") {
      console.log("priority must be a number, or high/low");
      return;
    }

    if (p < 0) {
      console.log("priority must be at least 0, or high/low");
      return;
    }
    // 注册组件其实就是把组件装入map里
    this.callback.set(component, callback);
    this.setComponentPriority(component, p);
    this.setComponentStatus(component, "pending");
  };
  unregisterComponent = (component) => {
    this.components.delete(component);
    this.status.delete(component);
    this.callback.delete(component);
  };
  componentDidRendered = (component) => {
    // 将当前组件的状态设为approval，表示可以渲染
    // 然后将下一个优先级的组件执行渲染
    this.setComponentStatus(component, "approval");
    const priority = this.getComponentPriority(component);
    const nextPriority = this.findNextPriority(priority);

    if (this.isAllRenderedOfPriority(priority)) {
      this.notifyRender(nextPriority);
    }
  };
  notifyRender = (priority) => {
    const p = this.convertPriority(priority);
    // 找到所有优先级为priority的组件，转换其状态，并进行渲染
    for (const [key, value] of this.components) {
      if (value === p) {
        this.setComponentStatus(key, "approval");

        try {
          const callback = this.callback.get(key);
          callback && callback();
        } catch (error) {
          console.log(error);
        }
      }
    }
  };
  allowRenderSelf = (component) => {
    // 判断是否可以渲染自己，判断依据是status
    // 还会判断上一个优先级的组件是否存在，如果还有上一个优先级的没渲染完，那么当前组件不会渲染
    if (this.getComponentStatus(component) === "approval") {
      return true;
    }

    const priority = this.getComponentPriority(component);

    if (priority === 0) {
      return true;
    }

    const previousPriority = this.findPreviousPriority(priority);

    if (!this.hasPriority(previousPriority)) {
      return false;
    }

    return this.isAllRenderedOfPriority(previousPriority);
  };
  setComponentStatus = (component, status) => {
    // 修改组件渲染状态，即pending或approval
    if (!this.components.has(component)) {
      return;
    }

    this.status.set(component, status);
  };
  getComponentStatus = (component) => {
    return this.status.get(component);
  };
  setComponentPriority = (component, priority) => {
    this.components.set(component, priority);
  };
  getComponentPriority = (component) => {
    return this.components.get(component);
  };
  isAllRenderedOfPriority = (priority) => {
    // 找到所有优先级为priority的组件，判断是否还有pending状态的（是否还有没渲染的）
    const p = this.convertPriority(priority);
    const s = [];
    this.components.forEach((value, key) => {
      if (value === p) {
        s.push(this.getComponentStatus(key));
      }
    });
    return !s.includes("pending");
  };
  findNextPriority = (priority) => {
    // 找到下一个优先级
    const p = this.convertPriority(priority);
    const sorted = Array.from(this.components.values()).sort();

    if (priority === undefined) {
      return sorted[0];
    }

    const index = sorted.lastIndexOf(p);

    if (+index >= 0) {
      return sorted[+index + 1];
    } else {
      return -1;
    }
  };
  findPreviousPriority = (priority) => {
    // 找到上一个优先级
    const p = this.convertPriority(priority);
    const sorted = Array.from(this.components.values()).sort();
    const index = sorted.indexOf(p);

    if (+index >= 1) {
      return sorted[+index - 1];
    } else {
      return -1;
    }
  };
  hasPriority = (priority) => {
    const p = this.convertPriority(priority);
    return [...this.components.values()].includes(p);
  };
}
```