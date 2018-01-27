```ts
import {
  Attribute, ChangeDetectorRef, ComponentFactoryResolver,
  ComponentRef, Directive, EventEmitter, Injector, OnDestroy,
  OnInit, Output, ViewContainerRef
} from '@angular/core';
import { ChildrenOutletContexts } from '../router_outlet_context';
import { ActivatedRoute } from '../router_state';
import { PRIMARY_OUTLET } from '../shared';

@Directive({ selector: 'router-outlet', exportAs: 'outlet' })
export class RouterOutlet implements OnDestroy, OnInit {
  private activated: ComponentRef<any> | null = null;
  private _activatedRoute: ActivatedRoute | null = null;
  private name: string;
  @Output('activate') activateEvents = new EventEmitter<any>();
  @Output('deactivate') deactivateEvents = new EventEmitter<any>();
  constructor(
    private parentContexts: ChildrenOutletContexts,
    private location: ViewContainerRef,
    private resolver: ComponentFactoryResolver,
    @Attribute('name') name: string,
    private changeDetector: ChangeDetectorRef
  ) {
    this.name = name || PRIMARY_OUTLET;
    parentContexts.onChildOutletCreated(this.name, this);
  }
  ngOnDestroy(): void { this.parentContexts.onChildOutletDestroyed(this.name); }
  ngOnInit(): void {
    if (!this.activated) {
      const context = this.parentContexts.getContext(this.name);
      if (context && context.route) {
        if (context.attachRef) {
          this.attach(context.attachRef, context.route);
        } else {
          this.activateWith(context.route, context.resolver || null);
        }
      }
    }
  }
  get isActivated(): boolean { return !!this.activated; }
  get component(): Object {
    if (!this.activated) throw new Error('Outlet is not activated');
    return this.activated.instance;
  }
  get activatedRoute(): ActivatedRoute {
    if (!this.activated) throw new Error('Outlet is not activated');
    return this._activatedRoute as ActivatedRoute;
  }
  get activatedRouteData() {
    if (this._activatedRoute) {
      return this._activatedRoute.snapshot.data;
    }
    return {};
  }
  detach(): ComponentRef<any> {
    if (!this.activated) throw new Error('Outlet is not activated');
    this.location.detach();
    const cmp = this.activated;
    this.activated = null;
    this._activatedRoute = null;
    return cmp;
  }
  attach(ref: ComponentRef<any>, activatedRoute: ActivatedRoute) {
    this.activated = ref;
    this._activatedRoute = activatedRoute;
    this.location.insert(ref.hostView);
  }
  deactivate(): void {
    if (this.activated) {
      const c = this.component;
      this.activated.destroy();
      this.activated = null;
      this._activatedRoute = null;
      this.deactivateEvents.emit(c);
    }
  }
  activateWith(activatedRoute: ActivatedRoute, resolver: ComponentFactoryResolver | null) {
    if (this.isActivated) {
      throw new Error('Cannot activate an already activated outlet');
    }
    this._activatedRoute = activatedRoute;
    const snapshot = activatedRoute._futureSnapshot;
    const component = <any>snapshot.routeConfig!.component;
    resolver = resolver || this.resolver;
    const factory = resolver.resolveComponentFactory(component);
    const childContexts = this.parentContexts.getOrCreateContext(this.name).children;
    const injector = new OutletInjector(activatedRoute, childContexts, this.location.injector);
    this.activated = this.location.createComponent(factory, this.location.length, injector);
    this.changeDetector.markForCheck();
    this.activateEvents.emit(this.activated.instance);
  }
}

class OutletInjector implements Injector {
  constructor(
    private route: ActivatedRoute,
    private childContexts: ChildrenOutletContexts,
    private parent: Injector
  ) { }

  get(token: any, notFoundValue?: any): any {
    if (token === ActivatedRoute) {
      return this.route;
    }
    if (token === ChildrenOutletContexts) {
      return this.childContexts;
    }
    return this.parent.get(token, notFoundValue);
  }
}
```