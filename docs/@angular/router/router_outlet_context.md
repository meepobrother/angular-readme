```ts
import { ComponentFactoryResolver, ComponentRef } from '@angular/core';
import { RouterOutlet } from './directives/router_outlet';
import { ActivatedRoute } from './router_state';
/**
 * outlet内容
 */
export class OutletContext {
  /**
   * router outlet
   */
  outlet: RouterOutlet | null = null;
  /**
   * router状态
   */
  route: ActivatedRoute | null = null;
  /**
   * 解析器
   */
  resolver: ComponentFactoryResolver | null = null;
  /**
   * 下级outlet内容
   */
  children = new ChildrenOutletContexts();
  /**
   * 绑定组件
   */
  attachRef: ComponentRef<any> | null = null;
}

export class ChildrenOutletContexts {
  // 内容
  private contexts = new Map<string, OutletContext>();
  // 创建时
  onChildOutletCreated(childName: string, outlet: RouterOutlet): void {
    const context = this.getOrCreateContext(childName);
    context.outlet = outlet;
    this.contexts.set(childName, context);
  }
  // 销毁时
  onChildOutletDestroyed(childName: string): void {
    const context = this.getContext(childName);
    if (context) {
      context.outlet = null;
    }
  }
  // 初始化
  onOutletDeactivated(): Map<string, OutletContext> {
    const contexts = this.contexts;
    this.contexts = new Map();
    return contexts;
  }
  // 重新挂载时
  onOutletReAttached(contexts: Map<string, OutletContext>) {
    this.contexts = contexts;
  }
  // 获取或者创建context
  getOrCreateContext(childName: string): OutletContext {
    let context = this.getContext(childName);
    if (!context) {
      context = new OutletContext();
      this.contexts.set(childName, context);
    }
    return context;
  }
  // 获取context
  getContext(childName: string): OutletContext | null {
    return this.contexts.get(childName) || null;
  }
}
```