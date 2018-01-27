```ts
import { ComponentFactoryResolver, ComponentRef } from '@angular/core';
import { RouterOutlet } from './directives/router_outlet';
import { ActivatedRoute } from './router_state';
export class OutletContext {
  outlet: RouterOutlet | null = null;
  route: ActivatedRoute | null = null;
  resolver: ComponentFactoryResolver | null = null;
  children = new ChildrenOutletContexts();
  attachRef: ComponentRef<any> | null = null;
}

export class ChildrenOutletContexts {
  private contexts = new Map<string, OutletContext>();
  onChildOutletCreated(childName: string, outlet: RouterOutlet): void {
    const context = this.getOrCreateContext(childName);
    context.outlet = outlet;
    this.contexts.set(childName, context);
  }
  onChildOutletDestroyed(childName: string): void {
    const context = this.getContext(childName);
    if (context) {
      context.outlet = null;
    }
  }
  onOutletDeactivated(): Map<string, OutletContext> {
    const contexts = this.contexts;
    this.contexts = new Map();
    return contexts;
  }
  onOutletReAttached(contexts: Map<string, OutletContext>) {
    this.contexts = contexts;
  }
  getOrCreateContext(childName: string): OutletContext {
    let context = this.getContext(childName);
    if (!context) {
      context = new OutletContext();
      this.contexts.set(childName, context);
    }
    return context;
  }
  getContext(childName: string): OutletContext | null {
    return this.contexts.get(childName) || null;
  }
}
```