# Android-MVC-MVP-MVVM
MVC MVP MVVM 
#  MVC/MVP/MVVM

## 前言

做移动端开发和前端开发的人员，对 MVC、MVP、MVVM 这几个名词应该都不陌生，这是三个最常用的应用架构模式，目的都是为了将业务和视图的实现代码分离，从而使同一个程序可以使用不同的表现形式。不过，网上的文章对这方面的解说众说纷纭，其中不乏有些错误的描述，导致有些人应用这些架构模式时陷入一些错误陷阱。本文将追根溯源，力求让大伙对这三个架构模式形成正确认识。

MVC = Model-View-**Controller**，MVP = Model-View-**Presenter**，MVVM = Model-View-**ViewModel**。这三个架构模式，都分别有三个不同的部件，都有相同的 **Model** 层和 **View** 层。Model 为模型层，主要管理业务模型的数据和行为；View 为展示层，其职责就是管理用户界面。三个架构模式目的都是为了解耦 Model 和 View，主要不同点就在于三者实现解耦的方案不同。Controller、Presenter、ViewModel，对应着三种不同的解耦方案，三种与 M 和 V 的连接方式。

下面，我们先从 MVC 开始，正确理解了 MVC，另外两个就非常容易理解了。

## MVC

MVC 最初被用于 Smalltalk-80，对 MVC 最早的解说来源于一篇论文《Applications Programming in Smalltalk-80(TM):How to use Model-View-Controller (MVC)》，那大概是在 1979 年的时候。该论文对 M-V-C 三个模块以及他们之间的通信都阐述了一些设计细节。

在 MVC 中，对应用程序划分出了三种角色：Model、View、Controller。三者有各自的具体用途和职责，并通过彼此的相互通信实现程序功能。对 MVC 了解不深的人其实存在疑惑，比如，Model 是否等于实体类？业务逻辑是归于 Controller 还是 Model？Model 与 View 具体是如何通信的？这些问题你会在下面的内容找到答案。

MVC 三件套中，最难理解的就是 Model，所以我们先来剖析它。前面我们说过，Model 层主要管理业务模型的数据和行为，它既保存程序的数据，也定义了处理该数据的逻辑，所以 **Model = 数据 + 业务逻辑**。因此，**处理业务逻辑属于 Model 的职责，而非 Controller**。从数据的维度来说，可以细分为**数据的定义、数据的存储、数据的获取**。数据的定义其实就是定义数据结构，一般用实体类来定义，以方便在不同角色间传递数据。数据的存储和获取则可能有几种途径：数据库、网络或缓存等。因此，在实际应用中，一个 Model 并不只是简单的一个对象，而是一个更广泛的层级。很多时候，会将 Model 层再进行分解，比如应用于客户端程序时，可以将 Model 层再细分为**业务逻辑层、网络层、存储层等**，而**实体类其实只是贯穿其中的一种数据结构而已**。不过，狭义上，当我们说一个 Model 对象的时候，主要是对外部组件而言的，更多是指这个 Model 对外所提供的数据，并不关心数据从何而来。这可能就是让很多人将 Model 误认为就是实体类的原因。

View 是 MVC 里最好理解的，它会接收用户的交互请求并展示数据信息给用户。一个 View 展示的数据可能只是一个 Model 对象的部分数据，也可能是一个 Model 对象的全部数据，甚至可能是多个 Model 对象数据的组合。在 MVC 里，View 被设计为可嵌套的，使用了**组合(Composite)模式**来实现。比如，列表视图(ListView)或表格视图(TableView)由每个 Item 组成，每个 Item 又可以由图片、文本、按钮等组成。View 是倾向于可复用的，因此，在实际应用中，倾向于将 View 开发成相对通用的组件。

Controller 层主要担任 Model 与 View 之间的桥梁，用于控制程序的流程。Controller 负责确保 View 可以访问到需要显示的 Model 对象数据，并充当 View 了解 Model 更改的渠道。View 接收到用户的交互请求之后，会将请求转发给 Controller，Controller 解析用户的请求之后，就会交给对应的 Model 去处理。因此，理论上，Controller 应该是很轻的。

## MVC 通信机制

其实，MVC 发展至今，三件套之间的通信机制，已经演化出多种通信路径。我们先来看看最初的版本：

![img](https://mmbiz.qpic.cn/mmbiz_png/Xibk1Sk7nmicms5E1PYVPyCbJ3cFRgtGSV6NGzZYlUoVicAAnuicbqF1ic0xdyspjMCWic7SowUnxPGmSO8GfrgCCf3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

该版本的 MVC 其实可以看成是由三个基本设计模式组合而成：**组合模式、策略模式、观察者模式**。组合模式前面已经讲过，就是用在了 View 层。View 与 Controller 之间则用了策略模式，Controller 对象为一个或多个 View 对象实现了策略，View 对象仅限于保持其视觉外观，而与程序逻辑相关的所有决策都委托给 Controller，即 View 可以使用不同的 Controller 实现，得到不同的行为。Model 与 View 之间则使用了观察者模式，View 会注册为 Model 的观察者，当 Model 有变化的时候，就能通知到 View。

所以，一般的交互流程就是：

1. 用户操纵 View，接着生成一个事件。比如用户点击某个按钮，则会生成一个点击事件。
2. Controller 对象接收事件并解释该事件，即，它应用了策略。该策略可以是请求 Model 对象以更改其状态，或请求 View 以更改其行为或外观。比如，一个注册按钮产生的事件被 Controller 接收之后，那它就会解释该事件，可能先校验用户的输入是否为空，如果为空则请求 View 提示让用户填写用户名和密码等；如果校验通过，那就请求 UserModel 创建新用户。
3. 当状态改变时，Model 对象又通知所有已注册为观察者的对象。如果观察者是 View 对象，则可以相应地更新其外观或行为。还是上面的例子，UserModel 创建新用户成功后，就可以通知观察者们，相应的 View 对象接收到 UserModel 创建新用户成功的通知后，就可以跳转到注册成功后的页面了。

这就是 MVC 最初版本的通信机制，自 1979 年提出之后就被广泛应用在 GUI 程序中。请注意，那时候还没有 HTTP。后来随着微软 ASP.NET MVC Framework 的出现，MVC 也开始被广泛应用于 Web 程序。

## 变种 MVC

以上版本的 MVC，由于 View 依赖了 Model，实际上减低了 View 的可复用性。那么，如果能将 View 和 Model 彻底解耦，那就可以提高 View 的可复用性了。因此，出现了下面这种变种 MVC：

![img](https://mmbiz.qpic.cn/mmbiz_png/Xibk1Sk7nmicms5E1PYVPyCbJ3cFRgtGSVpicZQsV24HyVh4GMZ9KZA32ugt0Gkf45HmiaMnY03sHbKcc6spOKQahA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

View 和 Model 不直接通信了，而统一通过 Controller 实现数据的传递。Model 将结果告知 Controller，Controller 再去更新 View。

据我所知，苹果提出了这种变种，在苹果之前，有没有其他人提出该变种，我不得而知。另外，很多 Web 框架的设计也是基于这种变种模式的 MVC 的设计思想，比如 SpringMVC 框架，当然，实际的实现比这个复杂得多，但主要设计思路还是 MVC。

这个变种，很多人会将其误认为另外一个经典架构模式「**三层架构**」，即他们认为 MVC 就是三层架构。其实，两者是不同的。三层架构分别为：**表现层、业务逻辑层、数据访问层**。虽然和 MVC 的通信方式很相似，但划分的各层的职责是不同的，最重要的是，两者的使用范围不同。三层架构是从整个应用程序架构的角度来划分的三层，而 MVC 只是表现层里再进行功能划分的设计方案，因此，要说两者之间有什么关联，那也是 MVC 属于三层架构里的一个子集。

接着，我们来看看在实际应用中的 MVC 结构又是怎样的？实际应用中，主要还是用在 App 开发上，以 iOS 为例，请看下图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



关键在于 **UIViewController**，其实，它不只是担任了 MVC 里的 Controller 角色，同时也承担了部分 View 的工作。我们可以将 View 角色按不同功能拆解成几个部分，一是负责界面的布局和渲染展示，二是负责界面的生命周期管理，三是负责界面数据的填充。这三部分功能，二和三其实都是由 UIViewController 负责的，即它不止承担 Controller 的角色，也承担了 View 的生命周期管理及 View 的数据管理。而独立的 View 就只剩下界面展示的功能，在 iOS 中主要就是 **xib 和 Storyboard** 充当这个角色。

**Android 的 Activity** 也是一样的，同时担任 Controller 和部分 View 的角色。

另外，在 App 应用里，Controller 从 Model 请求数据时，通常会比较耗时，所以 Model 会异步通知 Controller。

## MVC总结

先对 MVC 做一个小总结。MVC 为业务和视图的实现分离提供了开创性的设计思路，让负责业务逻辑的 Model 与负责展示的 View 实现了解耦，从而 Model 的复用性高，多个 View 就可以共享一个 Model，以及，在不修改 Model 的情况下就可以替换 View 的表现形式。

在交互上，早期的 MVC，View 是直接依赖于 Model 的，因此，View 的可复用性其实是受限制的。另外，这种模式其实也不适用于前后端分离的 Web 程序。因此，发展出了变种的 MVC，将 View 和 Model 的直接依赖切断，统一通过 Controller 进行调度，从而提高了 View 的可复用性，以及也可以将 MVC 扩展应用到前后端分离的 Web 程序。

不过，在 App 的实际应用中，又是另一种交互结构。出现了一个 ViewController 角色，不止承担 Controller 的职责，也承接了部分 View 的职责，主要包括对 View 的生命周期管理和数据填充等，而原本的 View 角色就只剩下展示的功能。这种方式的主要优点就是 View 变轻了，可复用性更高了；但缺点也很明显，原本的 Controller 是很轻的，但现在的 ViewController 则是很重的，承担了太多职责。

另外，在很多实际项目中，ViewController 这种模式其实还产生了副作用，当开发人员对 MVC 的理解不深的时候，就会错以为 MVC 的 Controller 就是这样子的，就会错将一些属于 View 和 Model 的代码也移到了 ViewController，导致已经很重的 ViewController 变得更臃肿了。因此，MVC 变成了 **Massive View Controller**，从而偏离了 MVC 原本的初衷。

为了从根本上解决这个问题，因此，很多 App 项目都改用 MVP 或 MVVM。接下来，我们再来看看 MVP 和 MVVM。

## MVP

对 MVP 模式最早的解说来源于一篇 1996 年时发表的论文：《MVP: Model-View-Presenter The Taligent Programming Model for C++ and Java》 。论文的作者是一个叫 Mike Potel 的人，当时是 Taligent 公司的 VP & CTO，Taligent 则是 IBM 的全资子公司。不过，如果你看过这篇论文的内容，就会发现，其实，最原始的 MVP 与我们现在所认识的 MVP 并不一样。论文中所论述的 MVP 其实是下面这样的结构：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

该 MVP 其实是从**数据管理**和**用户界面**两个维度的几个问题出发，将 Smalltalk 版本的 MVC 进行再分解演化而成，拆分出了几个中间组件：**Interactor、Commands、Selections**。Interactor 最好理解，其实就是 View 的交互事件。Commands 定义了对 Model 选定数据的一些操作，如删除、修改、保存等操作。Selections 可以理解为就是对 Model 数据集的筛选过滤，根据条件取子集。另外，所有的组件，也包括 Model、View、Presenter，都是通过接口进行交互的。

如果忽略掉三件套之间的几个中间组件，你就会发现，它们之间的依赖关系其实和 Smalltalk 版本的 MVC 是一样的。论文中也有提到，**Presenter 其实就是 Controller，只是为了与 MVC 区别开来，所以才称为 Presenter**。

再来看看我们现在所熟知的 MVP 结构图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

看到这关系图，你还会发现，这和前篇文章说的变种 MVC 不是一模一样吗？没错，关系图的确是一样的，但背后的实现和角色划分却不太一样，我们后面就讲。至于，这种模式的 MVP 是如何演化而来的，我也不得而知，只知道这已经成为了当代 MVP 的标准结构。

在 MVP 里，三件套各自的职责和依赖关系和变种 MVC 里的职责和依赖关系其实是一样的，但不同的是，MVP 之间的交互主要是通过接口实现的，Model、View、Presenter 都有各自的接口，定义各自的行为方法。针对接口编程，自然就能减低耦合，提高可复用性，以及容易进行单元测试。

之前我们说过，实际应用中的 MVC，**UIViewController** 和 **Activity** 其实是同时担任了 Controller 和部分 View 的角色的，职责划分不明确，才导致 UIViewController 和 Activity 的代码混乱不堪，越来越臃肿。而采用 MVP 模式，UIViewController 和 Activity 就明确地划分为 View 角色了，原有的 Controller 角色的职责则交由 Presenter 负责，职责清晰了，代码自然也容易清晰了。

## MVP 的简单使用

我们就以一个简单的登录案例来说明如何使用 MVP，下图是该案例的类图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

定义了 4 个接口和 3 个实现类，其中，**LoginActivity** 是 Android 的 Activity 类，在 iOS 中，则可定义为 **LoginViewController**。LoginActivity 实现了 LoginView 接口，同时还会持有一个 LoginPresenter 对象。LoginView 和 LoginActivity 都明确划分到 View 层，**LoginView** 定义了登录流程中涉及到的几个UI层的接口方法，包括显示和隐藏加载框，以及登录失败时的错误信息展示，和登录成功后的处理，这些方法都会在**LoginPresenterImpl** 的方法中被调用。Presenter 层定义了两个接口，LoginPresenter 只有一个登录的接口方法，**OnLoginFinishedListener** 则定义了登录结果的回调方法，两个接口统一由 LoginPresenterImpl 来实现即可。另外，从图中也可看到，LoginPresenterImpl 既持有一个 LoginView 对象，也持有一个 LoginModel 对象，LoginPresenterImpl 其实就是 LoginView 和 LoginModel 之间交互的桥梁。而 Model 层的 LoginModel 则不直接持有 LoginPresenter 对象，只在登录接口中加了一个回调对象的参数。

#### LoginActivity

下面，我们来简单看看一些关键代码的实现。先来看看 LoginActivity 的关键代码：

```
public class LoginActivity extends AppCompatActivity implements LoginView {
    private LoginPresenter loginPresenter;
   private ProgressBar progressBar;
   private LoginButton loginBtn;
   private EditText userNameEdt;
   private EditText passwordEdt;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
       progressBar = findViewById(R.id.loading);
       loginBtn = findViewById(R.id.login);
       userNameEdt = findViewById(R.id.username);
       passwordEdt = findViewById(R.id.password);
      
        loginPresenter = new LoginPresenterImpl();
       loginPresenter.attachView(this);
      
       loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loginPresenter.login(userNameEdt.getText().toString(),
                        passwordEdt.getText().toString());
            }
        });
    }
  
   @Override
    public void showLoading() {
        if (!progressBar.isShowing()) {
            progressBar.show();
        }
    }
  
    @Override
    public void hideLoading() {
        if (progressBar.isShowing()) {
            progressBar.dismiss();
        }
    }
    
    @Override
    public void onLoginError(String msg) {
        Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
    }
  
   @Override
    public void onLoginSuccess() {
        Toast.makeText(this, "Success", Toast.LENGTH_SHORT).show();
       startActivity(new Intent(this, MainActivity.class));
    }
  
   @Override
    protected void onDestroy() {
        super.onDestroy();
        loginPresenter.detachView();
    }
}
```

代码逻辑比较简单，从代码中可知，在 onCreate() 方法里初始化了 loginPresenter 对象，并绑定了自身作为 LoginView 的引用，并在登录按钮的点击事件中调用了 loginPresenter.login() 方法，从而将登录事件传递给了 loginPresenter。

#### LoginPresenterImpl

Presenter 就是 View 和 Model 之间的桥梁，我们来看看 LoginPresenterImpl 的关键代码：

```
public class LoginPresenterImpl implements LoginPresenter, OnLoginFinishedListener {
   private LoginView loginView;
   private LoginModel loginModel;
  
   public LoginPresenterImpl() {
       this.loginModel = new LoginModelImpl();
    }
  
   @Override
   public void attachView(LoginView loginView) {
        this.loginView = loginView;
    }
    
   @Override
    public void detachView() {
        this.loginView = null;
    }
    
   @Override
    public boolean isViewAttached(){
        return loginView != null;
    }
   
   @Override
    public void login(String username, String password) {
       loginView.showLoading();
       loginModel.login(username, password, this);
    }
  
   @Override
    public void onLoginError(String msg) {
       if (isViewAttached()) {
           loginView.hideLoading();
         loginView.onLoginError(msg);
        }
    }
  
   @Override
    public void onLoginSuccess() {
       if (isViewAttached()) {
           loginView.hideLoading();
         loginView.onLoginSuccess();
        }
    }
}
```

#### LoginModelImpl

最后，LoginModelImpl 就是对登录的业务逻辑处理了，一般是通过网络请求服务器实现登录逻辑，我们来看看伪代码：

```
public class LoginModelImpl implements LoginModel {
   @Override
   public void login(String username, String password, OnLoginFinishedListener listener) {
       apiService.login(username, password)
           .subscribeOn(Schedulers.newThread())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe(new Subscriber<Response>() {
              @Override
              public void onCompleted() {
              }

              @Override
              public void onError(Throwable e) {
                 listener.onLoginError("Server Error")
              }

              @Override
              public void onNext(Response res) {
                 if (res.isSuccess()) {
                       listener.onLoginSuccess();
                    } else {
                       listener.onLoginError(res.getMsg());
                    }
              }
           });
    }
}
```

该示例代码用到了 Retrofit + RxJava，最核心的代码就在 onNext() 方法里，这是请求响应返回后被调用的方法。

至此，MVP 最简单的使用案例就讲解完了。另外，由于每一个业务功能基本都有对应的一套 MVP 的接口，因此，很多时候会将这一套接口封装在一起，组成一套契约，如下所示：

```
public interface LoginContract {
   public interface LoginView {
       ...
    }
  
   public interface LoginPresenter {
       ...
    }
  
   public interface OnLoginFinishedListener {
       ...
    }
  
   public interface LoginModel {
       ...
    }
}
```

至此，对 MVP 最简单的用法就演示到这里结束。在实际应用中，应该做一些调整，比如抽离出一个 Base 层，将一些冗余的重复代码封装到对应的 Base 接口或类中，比如抽离出 BaseView、BaseActivity、BasePresenter、BaseListener 等。以及，设计上每一套 MVP 尽量设计成实现单一功能，以便能被复用，从而无需每一个页面都编写单独一套 MVP，尽量使用组合或继承的方式复用 MVP。

## MVP 的优缺点

接着，我们来总结下 MVP 有哪些优缺点。先来看看有哪些优点呢：

1. 我们知道，MVC 模式在 App 实际应用中，Activity 和 UIViewController 既同时担任 Controller 又担任部分 View 的职责，职责不清，导致 Activity 和 UIViewController 容易变得越来越臃肿。而应用 MVP 模式，直接将 Activity 和 UIViewController 划分到 View 层了，职责明确了，自然也避免了 Activity 和 UIViewController 臃肿的问题。
2. MVP 之间的交互通过接口来进行的，那就便于进行单元测试了，维护性和扩展性也提高了。
3. M 和 V 之间彻底分离了，降低了耦合性，修改 V 层也不会影响 M 层。

不过，相应地，相比 MVC 也引入了一些弊端：

1. 由于增加了很多接口的定义，需要编写的代码量暴增，增加了项目的复杂度。
2. 需要对很多业务模块之间的交互抽象成接口定义，对开发人员的设计能力要求更高了。

## MVVM

MVVM = Model-View-ViewModel，与 MVC、MVP 不同的就在于最后一个部件，换成了**ViewModel**（VM）。MVVM 最早于 2005 年被微软的 WPF 和 Silverlight 的架构师 John Gossman 提出，并且应用在微软的软件开发中。John Gossman 当年写的那篇介绍文章为《Introduction to Model/View/ViewModel pattern for building WPF apps》，内容比较简单。另外，为了方便不熟悉英文的小伙伴们，我还找到了一篇译文。

MVVM 的关系图如下：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



可看出，MVVM 的关系图和 MVP 很相似，最大的不同在于 View 和 ViewModel 之间主要是通过数据绑定的方案来实现交互的。

要理解 MVVM，我们先来理解什么是 ViewModel？和 Model 有什么区别？

#### ViewModel

Model 封装了**业务逻辑和数据**，管理的是业务模型。而 **ViewModel = Model of View**，即视图的模型，封装的是**视图的表示逻辑和数据，是对视图的抽象，包括视图的属性和命令，或视图的状态和行为**。

我们讲一个示例吧，就比如我们要在页面中展示一个购物订单的信息，视图中要展示的内容包括订单号、订单状态、创建时间、成交时间、订单金额、商品名称、购买数量等。那么，在业务模型中，该订单的数据结构一般是这样的：

```
{
    "orderId": "107896581204552",
    "status": "paid",
    "createdTime": 1592826572000,
    "dealTime": 1592826691000,
    "amount": 105.28,
    "productName": "VicTsing MM057 2.4G 无线便携式光学鼠标，带 USB 接收器",
    "buyQty": 1
}
```

而要将这个订单的数据展示到界面，肯定还需要做一些数据转换，比如，需要将时间戳转换为「yyyy-MM-dd HH:mm:ss」格式的时间，paid 转换为「已付款」，金额前面再加个￥符号，数量前面加多个乘号。所以，对应到界面上的数据结构大概是这样的：

```
{
    "orderId": "107896581204552",
    "status": "已付款",
    "createdTime": "2020-06-22 19:49:32",
    "dealTime": "2020-06-22 19:51:31",
    "amount": "￥105.28",
    "productName": "VicTsing MM057 2.4G 无线便携式光学鼠标，带 USB 接收器",
    "buyQty": "X1"
}
```

这就是 ViewModel 中所指的视图数据，不过这里演示的只是视图的属性，也是视图状态。但 ViewModel 封装的除了属性，也包括命令，即视图行为，比如页面刚加载进来时发生什么，点击某个按钮发生什么，点击列表中的某个 item 又发生什么，这些都属于视图行为。

#### 数据绑定

MVVM 最重要的一个特性就是数据绑定，通过将 View 的属性绑定到 ViewModel，可以使两者之间松耦合，也完全不需要在 ViewModel 里写代码去直接更新一个 View。数据绑定系统还支持输入验证，这提供了将验证错误传输到 View 的标准化方法。

通过数据绑定，当 ViewModel 的数据发生改变之后，与之绑定的 View 也会随之自动更新。反过来，如果 View 发生了变化，那 ViewModel 是否也同样会随之变化呢？这就涉及到数据绑定的两种类型：

**单向绑定**：ViewModel 与 View 绑定之后，ViewModel 变化后，View 会自动更新，但反之不然，即数据传递的方向是单向的。**（ViewModel —> View）**

**双向绑定**：ViewModel 与 View 绑定之后，如果 View 和 ViewModel 中的任何一方变化后，另一方都会自动更新，这就是双向绑定。（**Model <—> View**）

一般情况下，在视图中只显示而无需编辑的数据用单向绑定，需要编辑的数据才用双向绑定。

前面我们已经了解到，ViewModel 封装的数据包含 View 的属性和命令两种，因此，数据绑定其实也可分为**属性绑定**和**命令绑定**。比如，TextView 的内容绑定的就是属性，Button 的点击事件绑定的就是命令。

要实现数据绑定，通常都采用发布者-订阅者模式进行实现，但这部分工作如果由开发人员自己来写代码实现，其实还是挺复杂的，因此，各大平台都提供了各自的内部实现。比如 Vue 和 React 自身都实现了数据绑定，Android 目前最主流的方案就是采用 Jetpack，iOS 最常用的方案则是结合 ReactiveCocoa（RAC）实现。

## MVVM 的使用

我们重点讲解下如何用 Jetpack 实现 MVVM 架构，Jetpack 提供了多个架构组件，包括 ViewModel、LiveData、DataBinding 等，Android 官方推荐的应用架构如下图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

该架构对应于 MVVM 的话，则 Activity/Fragment 属于 View 层，Repository 和下面的 Model 及 Remote Data Source 则为 MVVM 中的 Model 层。该架构图没提到 DataBinding，但我们会使用到。我们将用 DataBinding、ViewModel、LiveData 三者结合来实现数据绑定的需求。至于这几个组件的基本用法，我就不详细讲解了，不熟悉的自行去学习下即可。

我们还是以登录页面为例，我们页面将展示4个控件：登录账号的输入框、密码的输入框、登录按钮、登录成功后返回的 UID。用户输入登录账号和密码之后，点击登录按钮，将向服务器发送登录请求，登录成功后会返回 UID，最后将 UID 展示到页面上。而登录账号则会缓存到本地数据库，第二次打开页面则会从缓存中读取并直接展示登录账号，无需再输入。

#### LoginRepository

我们先来讲讲 Model 层，Model 层主要需要实现两个功能：一是通过网络请求实现登录，登录成功后会得到 UID；二是将登录账号保存到 SQLite，实现登录账号的缓存功能。为了简便，我们不编写完整代码，就只用伪代码简单模拟一下实现流程：

```
public class LoginRepository {
   ...
   public String login(String userName, String password) {
       // 调用网络请求
       String uid = apiService.login(userName, password);
        // 将userName缓存到本地数据库
        cache.save("userName", userName);
      
        return uid;
    }
   // 从缓存中获取登录账号
   public String getUserNameFromCache() {
       return cache.get("userName");
    }
}
```

代码很简单，**login()** 方法接收两个参数：用户名和密码，调用登录接口返回 uid，接着将用户名缓存起来，最后返回 uid。**getUserNameFromCache()** 方法的逻辑更简单，就直接从缓存中读取并返回。

#### LoginViewModel

接着，再来聊聊 ViewModel，这个就很关键了。我们先来看看代码：

```
public class LoginViewModel extends ViewModel {
    public MutableLiveData<String> userName;
    public MutableLiveData<String> password;
    public MutableLiveData<String> uid;

    private LoginRepository repository;
  
    public LoginViewModel() {
        repository = new LoginRepository();
        userName.postValue(repository.getUserNameFromCache());
        password = new MutableLiveData<>("");
        uid = new MutableLiveData<>("0");
    }

    public void login() {
        String userId = repository.login(userName.getValue(), password.getValue());
        uid.postValue(userId);
    }
}
```

虽然代码量也比较少，但里面的信息比较多，且听我一一道说。

首先，我们继承了 ViewModel，这是 Jetpack 提供的组件，其用途是封装**界面控制器**（如 Activity 和 Fragment）的数据，以使数据在配置更改后仍然存在。直白的意思就是，将界面和数据进行分离，这也是 MVVM 的一个关键思想。如果不分离，假如旋转了设备屏幕，那需要我们自己去做数据保存和恢复的工作。分离之后，数据就不会受界面控制器这些中间状态的生命周期而影响。

其次，你会发现，LoginViewModel 并没有 Activity 或 Fragment 的引用，也没有像 MVP 所定义的 LoginView 的接口实例的引用。即是说，ViewModel 并不依赖于 View 层，所以，ViewModel 自然也更便于测试和复用。

然后，我们将 **userName、password、uid** 三个变量声明为 **MutableLiveData** 类型，就可以实现自动将数据变化通知给界面。因此，上面代码中，数据变化后我们并没有再添加代码去通知界面更新 UI，其背后的机制已经自动帮我们完成了通知。

最后，LoginViewModel 与 View 界面进行了绑定的除了 userName、password、uid 这三个属性之外，其实还有一个**命令绑定**，就是 **login()** 方法，其绑定了按钮的点击事件。

#### LoginActivity

最后，就是 View 层了。LoginActivity 的核心代码很简单，代码如下：

```
public class LoginActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    // 初始化viewModel
        LoginViewModel viewModel = new ViewModelProvider(this).get(LoginViewModel.class);
    // 与布局文件进行绑定
        ActivityLoginBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_login);
        binding.setLifecycleOwner(this);
        binding.setViewmodel(viewModel);
    }
}
```

以上几行代码就实现了 viewModel 和布局文件绑定的功能，很简单吧。Jetpack 本身帮我们实现了背后的复杂逻辑，所以我们就可以很简单的实现功能。

布局文件 activity_login.xml 则如下：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="vm"
            type="com.example.mvvm.LoginViewModel" />
    </data>
    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <EditText
            android:id="@+id/userName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@={vm.userName}"
            app:layout_constraintBottom_toTopOf="@+id/password"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />
        <EditText
            android:id="@+id/password"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:inputType="textPassword"
            android:text="@={vm.password}"
            app:layout_constraintBottom_toTopOf="@+id/login"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/userName" />
        <Button
            android:id="@+id/login"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{() -> vm.login()}"
            android:text="Login"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/password" />
        <TextView
            android:id="@+id/uid"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{vm.uid}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/login" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

其中，data 内的 variable 标签定义的就是我们的 LoginViewModel，我将其命名为 vm，然后就可以在下面的控件中引用它。

来看看两个 EditText 和 最后的 TextView 的 android:text 属性的值是怎么设置的？代码中分别设置为了 **@={vm.userName}、@={vm.password}、@{vm.uid}**，可以看出对应的正是 LoginViewModel 的三个属性。设置时，如果@后面不加等号，那就只是单向绑定，只能由 ViewModel 将数据变化通知到界面。加了等号，才是双向绑定，即界面上的数据改变才能传递给到 ViewModel。

再看看 Button 的 android:onClick 属性值，设置为了 **@{() -> vm.login()}**，这就是将该按钮的点击事件绑定到 ViewModel 的 login() 方法的一种写法。

至此，MVVM 的使用就讲解到这里。

## 总结

总结一下，MVP 和 MVVM 都是为了解决界面和数据的分离问题，两者只是采用了不同的实现方案。MVP 之间的交互主要是通过接口实现的，其主要弊端就是需要编写大量接口。而 MVVM 则是通过数据绑定的方式实现交互，虽然其实现需要依赖具体的一些框架工具，但明显大大减少了开发者需要编写的代码量。
