Shsqa Android App

Android Google Play Consumer Application for Argos' APIs.


Using Dagger (dependency injection)

This project uses Dagger for Java and Android.


Scopes

There are three main scopes in this application:

#### 1. Application
 Objects at this scope are injectable everywhere (application, activities and fragments), The @Singleton annotation can be utilised such that only a single instance of an object is maintained and injected.

#### 3. Activity
Objects at this scope are created per activity and are destroyed if the activity is destroyed. Objects at this scope can be injected to the activity in question and any fragments it hosts.

If you're creating an "injectable" activity, it must either extend BaseActivity, DaggerActivity or DaggerAppCompatActivity. It must also be declared in the ActivityBuilder class with an abstract binding method:

@ActivityScope
@ContributesAndroidInjector(modules = {TrolleyModule.class, TrolleyFragmentProvider.class})
abstract TrolleyActivity provideTrolleyActivity();
In the example above, TrolleyModule is providing activity-level dependencies and is delegating fragment dependencies to TrolleyFragmentProvider.

#### 4. Fragment
Objects at this scope are created per fragment and are destroyed if the fragment is destroyed.

If you're creating an "injectable" fragment, it must either extend BaseFragment, DaggerFragment or DaggerFragment (support). It must also have an activity that is also "injectable".

The parent activity needs to have a fragment provider module listed as a module in ActivityBuilder (see above). This provider module should contain abstract method bindings for providing dependencies for the fragments this activity will host.

@Module
public abstract class TrolleyFragmentProvider {

    @ContributesAndroidInjector(modules = TrolleyContainerFragmentProvider.class)
    abstract TrolleyContainerFragment provideTrolleyContainerFragment();

    @ContributesAndroidInjector(modules = {TrolleyEmptyModule.class, TrolleyEmptyFragmentProvider.class})
    abstract TrolleyEmptyFragment provideTrolleyEmptyFragment();

}
For example, TrolleyContainerFragment has a child fragment, so it once again delegates it to TrolleyContainerFragmentProvider. However TrolleyEmptyFragment needs TrolleyEmptyModule but also a child fragment provider TrolleyEmptyFragmentProvider. We can use this to reduce the scope of injectable objects to where it is only needed.


Provider methods

Modules contain provider methods such as

@Provides
@Singleton
@SubscribeOnSchedulerQualifier Scheduler provideSubscribeOnScheduler() {
    return Schedulers.io();
}
We want to let Dagger resolve dependencies on its own, as that makes these objects flexible for construction. The following is an example of what we should avoid doing:

@Provides TrolleyListMvp.Presenter provideTrolleyListPresenter(final TrolleyRepo trolleyRepo,
                                                               @SubscribeOnSchedulerQualifier final Scheduler subscribeOnScheduler,
                                                               @ObserveOnSchedulerQualifier final Scheduler observeOnScheduler,
                                                               final TrolleyFulfilmentChangeEvent trolleyFulfilmentModeEvent,
                                                               final RxEventDebouncer<ChangeQuantityEvent> rxEventDebouncer,
                                                               final Logger androidLogger) {
    return new TrolleyListPresenter(trolleyRepo,
            subscribeOnScheduler,
            observeOnScheduler,
            trolleyFulfilmentModeEvent,
            rxEventDebouncer,
            androidLogger);
}
Instead we should be doing the following and create provider methods for it's dependencies.

@Provides TrolleyListMvp.Presenter provideTrolleyListPresenter(TrolleyListPresenter trolleyListPresenter) {
    return trolleyListPresenter;
}