<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ff7dc-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="ff7dc-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="ff7dc-103">Для этого необходимо интегрировать [библиотеку проверки подлинности Microsoft (MSAL) для iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) в приложение.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="ff7dc-104">Создайте новый файл со **списком свойств** в проекте **Графтуториал** с именем **ауссеттингс. plist**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="ff7dc-105">Добавьте следующие элементы в файл **корневого** словаря.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="ff7dc-106">Key</span><span class="sxs-lookup"><span data-stu-id="ff7dc-106">Key</span></span> | <span data-ttu-id="ff7dc-107">Тип</span><span class="sxs-lookup"><span data-stu-id="ff7dc-107">Type</span></span> | <span data-ttu-id="ff7dc-108">Значение</span><span class="sxs-lookup"><span data-stu-id="ff7dc-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="ff7dc-109">String</span><span class="sxs-lookup"><span data-stu-id="ff7dc-109">String</span></span> | <span data-ttu-id="ff7dc-110">Идентификатор приложения на портале Azure</span><span class="sxs-lookup"><span data-stu-id="ff7dc-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="ff7dc-111">Массив</span><span class="sxs-lookup"><span data-stu-id="ff7dc-111">Array</span></span> | <span data-ttu-id="ff7dc-112">Два строковых значения: `User.Read` и`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="ff7dc-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Снимок экрана: файл Ауссеттингс. plist в Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="ff7dc-114">Если вы используете систему управления версиями (например, Git), то теперь будет полезно исключить файл **ауссеттингс. plist** из системы управления версиями, чтобы избежать случайной утечки идентификатора приложения.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="ff7dc-115">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="ff7dc-115">Implement sign-in</span></span>

<span data-ttu-id="ff7dc-116">В этом разделе описывается настройка проекта для MSAL, создание класса диспетчера проверки подлинности и обновление приложения для входа и выхода.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="ff7dc-117">Настройка проекта для MSAL</span><span class="sxs-lookup"><span data-stu-id="ff7dc-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="ff7dc-118">Добавьте новую группу цепочки ключей в возможности проекта.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="ff7dc-119">Выберите проект **графтуториал** , а затем **подписывать & возможности**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="ff7dc-120">Выберите пункт **+ возможность**, а затем дважды щелкните **общий доступ к цепочке ключей**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="ff7dc-121">Добавьте группу цепочки ключей со значением `com.microsoft.adalcache`.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="ff7dc-122">Нажмите кнопку **info. plist** и выберите **Открыть как**, а затем **Исходный код**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="ff7dc-123">Добавьте следующий `<dict>` элемент в элемент.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-123">Add the following inside the `<dict>` element.</span></span>

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. <span data-ttu-id="ff7dc-124">Откройте **аппделегате. m** и добавьте приведенный ниже оператор import в начало файла.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-124">Open **AppDelegate.m** and add the following import statement at the top of the file.</span></span>

    ```objc
    #import <MSAL/MSAL.h>
    ```

1. <span data-ttu-id="ff7dc-125">Добавьте к классу `AppDelegate` следующую функцию:</span><span class="sxs-lookup"><span data-stu-id="ff7dc-125">Add the following function to the `AppDelegate` class.</span></span>

    ```objc
    - (BOOL)application:(UIApplication *)app
                openURL:(NSURL *)url
                options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url
                                             sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="ff7dc-126">Создание диспетчера проверки подлинности</span><span class="sxs-lookup"><span data-stu-id="ff7dc-126">Create authentication manager</span></span>

1. <span data-ttu-id="ff7dc-127">Создайте новый **класс Touch Cocoa** в проекте **Графтуториал** с именем **AuthenticationManager**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-127">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **AuthenticationManager**.</span></span> <span data-ttu-id="ff7dc-128">Выберите **NSObject** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-128">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="ff7dc-129">Откройте **AuthenticationManager. h** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-129">Open **AuthenticationManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSAL/MSAL.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetTokenCompletionBlock)(NSString* _Nullable accessToken, NSError* _Nullable error);

    @interface AuthenticationManager : NSObject<MSAuthenticationProvider>

    + (id) instance;
    - (void) getTokenInteractivelyWithParentView: (UIViewController*) parentView andCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) getTokenSilentlyWithCompletionBlock: (GetTokenCompletionBlock)completionBlock;
    - (void) signOut;
    - (void) getAccessTokenForProviderOptions:(id<MSAuthenticationProviderOptions>)authProviderOptions andCompletion:(void (^)(NSString *, NSError *))completion;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="ff7dc-130">Откройте **AuthenticationManager. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-130">Open **AuthenticationManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "AuthenticationManager.h"

    @interface AuthenticationManager()

    @property NSString* appId;
    @property NSArray<NSString*>* graphScopes;
    @property MSALPublicClientApplication* publicClient;

    @end

    @implementation AuthenticationManager

    + (id) instance {
        static AuthenticationManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Get app ID and scopes from AuthSettings.plist
            NSString* authConfigPath =
            [NSBundle.mainBundle pathForResource:@"AuthSettings" ofType:@"plist"];
            NSDictionary* authConfig = [NSDictionary dictionaryWithContentsOfFile:authConfigPath];

            self.appId = authConfig[@"AppId"];
            self.graphScopes = authConfig[@"GraphScopes"];

            // Create the MSAL client
            self.publicClient = [[MSALPublicClientApplication alloc] initWithClientId:self.appId error:nil];
        }

        return self;
    }

    - (void) getAccessTokenForProviderOptions:(id<MSAuthenticationProviderOptions>)authProviderOptions andCompletion:(void (^)(NSString * _Nonnull, NSError * _Nonnull))completion {
        [self getTokenSilentlyWithCompletionBlock:completion];
    }

    - (void) getTokenInteractivelyWithParentView:(UIViewController *)parentView andCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        MSALWebviewParameters* webParameters = [[MSALWebviewParameters alloc] initWithParentViewController:parentView];
        MSALInteractiveTokenParameters* interactiveParameters =
        [[MSALInteractiveTokenParameters alloc]initWithScopes:self.graphScopes webviewParameters:webParameters];

        // Call acquireToken to open a browser so the user can sign in
        [self.publicClient
         acquireTokenWithParameters:interactiveParameters
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {

            // Check error
            if (error) {
                completionBlock(nil, error);
                return;
            }

            // Check result
            if (!result) {
                NSMutableDictionary* details = [NSMutableDictionary dictionary];
                [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                return;
            }

            NSLog(@"Got token interactively: %@", result.accessToken);
            completionBlock(result.accessToken, nil);
        }];
    }

    - (void) getTokenSilentlyWithCompletionBlock:(GetTokenCompletionBlock)completionBlock {
        // Check if there is an account in the cache
        NSError* msalError;
        MSALAccount* account = [self.publicClient allAccounts:&msalError].firstObject;

        if (msalError || !account) {
            NSMutableDictionary* details = [NSMutableDictionary dictionary];
            [details setValue:@"Could not retrieve account from cache" forKey:NSDebugDescriptionErrorKey];
            completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
            return;
        }

        MSALSilentTokenParameters* silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:self.graphScopes
                                                                                                account:account];

        // Attempt to get token silently
        [self.publicClient
         acquireTokenSilentWithParameters:silentParameters
         completionBlock:^(MSALResult * _Nullable result, NSError * _Nullable error) {
             // Check error
             if (error) {
                 completionBlock(nil, error);
                 return;
             }

             // Check result
             if (!result) {
                 NSMutableDictionary* details = [NSMutableDictionary dictionary];
                 [details setValue:@"No result was returned" forKey:NSDebugDescriptionErrorKey];
                 completionBlock(nil, [NSError errorWithDomain:@"AuthenticationManager" code:0 userInfo:details]);
                 return;
             }

             NSLog(@"Got token silently: %@", result.accessToken);
             completionBlock(result.accessToken, nil);
         }];
    }

    - (void) signOut {
        NSError* msalError;
        NSArray* accounts = [self.publicClient allAccounts:&msalError];

        if (msalError) {
            NSLog(@"Error getting accounts from cache: %@", msalError.debugDescription);
            return;
        }

        for (id account in accounts) {
            [self.publicClient removeAccount:account error:nil];
        }
    }

    @end
    ```

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="ff7dc-131">Добавление входа и выхода</span><span class="sxs-lookup"><span data-stu-id="ff7dc-131">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="ff7dc-132">Откройте файл **сигнинвиевконтроллер. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-132">Open the **SignInViewController.m** file and replace its contents with the following code.</span></span>

    ```objc
    #import "SignInViewController.h"
    #import "SpinnerViewController.h"
    #import "AuthenticationManager.h"

    @interface SignInViewController ()
    @property SpinnerViewController* spinner;
    @end

    @implementation SignInViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenSilentlyWithCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // If there is no token or if there's an error,
                     // no user is signed in, so stay here
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
        }];
    }

    - (IBAction)signIn {
        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        [AuthenticationManager.instance
         getTokenInteractivelyWithParentView:self
         andCompletionBlock:^(NSString * _Nullable accessToken, NSError * _Nullable error) {
             dispatch_async(dispatch_get_main_queue(), ^{
                 [self.spinner stop];

                 if (error || !accessToken) {
                     // Show the error and stay on the sign-in page
                     UIAlertController* alert = [UIAlertController
                                                 alertControllerWithTitle:@"Error signing in"
                                                 message:error.debugDescription
                                                 preferredStyle:UIAlertControllerStyleAlert];

                     UIAlertAction* okButton = [UIAlertAction
                                                actionWithTitle:@"OK"
                                                style:UIAlertActionStyleDefault
                                                handler:nil];

                     [alert addAction:okButton];
                     [self presentViewController:alert animated:true completion:nil];
                     return;
                 }

                 // Since we got a token, user is signed in
                 // Go to welcome page
                 [self performSegueWithIdentifier: @"userSignedIn" sender: nil];
             });
         }];
    }
    @end
    ```

1. <span data-ttu-id="ff7dc-133">Откройте **велкомевиевконтроллер. m** и замените существующую `signOut` функцию на приведенную ниже.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-133">Open **WelcomeViewController.m** and replace the existing `signOut` function with the following.</span></span>

    ```objc
    - (IBAction)signOut {
        [AuthenticationManager.instance signOut];
        [self performSegueWithIdentifier: @"userSignedOut" sender: nil];
    }
    ```

1. <span data-ttu-id="ff7dc-134">Сохраните изменения и перезапустите приложение в симуляторе.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-134">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="ff7dc-135">Если вы входите в приложение, в окне вывода в Xcode должен отображаться маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-135">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Снимок экрана: окно вывода в Xcode, в котором отображается маркер доступа](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="ff7dc-137">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="ff7dc-137">Get user details</span></span>

<span data-ttu-id="ff7dc-138">В этом разделе описывается создание вспомогательного класса для хранения всех вызовов Microsoft Graph и обновление `WelcomeViewController` для использования этого нового класса для получения пользователя, выполнившего вход в систему.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-138">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="ff7dc-139">Создайте новый **класс Touch Cocoa** в проекте **Графтуториал** с именем **графманажер**.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-139">Create a new **Cocoa Touch Class** in the **GraphTutorial** project named **GraphManager**.</span></span> <span data-ttu-id="ff7dc-140">Выберите **NSObject** в **подклассе** поля.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-140">Choose **NSObject** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="ff7dc-141">Откройте **графманажер. h** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-141">Open **GraphManager.h** and replace its contents with the following code.</span></span>

    ```objc
    #import <Foundation/Foundation.h>
    #import <MSGraphClientSDK/MSGraphClientSDK.h>
    #import <MSGraphClientModels/MSGraphClientModels.h>
    #import "AuthenticationManager.h"

    NS_ASSUME_NONNULL_BEGIN

    typedef void (^GetMeCompletionBlock)(MSGraphUser* _Nullable user, NSError* _Nullable error);

    @interface GraphManager : NSObject

    + (id) instance;
    - (void) getMeWithCompletionBlock: (GetMeCompletionBlock)completionBlock;

    @end

    NS_ASSUME_NONNULL_END
    ```

1. <span data-ttu-id="ff7dc-142">Откройте **графманажер. m** и замените его содержимое приведенным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-142">Open **GraphManager.m** and replace its contents with the following code.</span></span>

    ```objc
    #import "GraphManager.h"

    @interface GraphManager()

    @property MSHTTPClient* graphClient;

    @end

    @implementation GraphManager

    + (id) instance {
        static GraphManager *singleInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^ {
            singleInstance = [[self alloc] init];
        });

        return singleInstance;
    }

    - (id) init {
        if (self = [super init]) {
            // Create the Graph client
            self.graphClient = [MSClientFactory
                                createHTTPClientWithAuthenticationProvider:AuthenticationManager.instance];
        }

        return self;
    }

    - (void) getMeWithCompletionBlock:(GetMeCompletionBlock)completionBlock {
        // GET /me
        NSString* meUrlString = [NSString stringWithFormat:@"%@/me", MSGraphBaseURL];
        NSURL* meUrl = [[NSURL alloc] initWithString:meUrlString];
        NSMutableURLRequest* meRequest = [[NSMutableURLRequest alloc] initWithURL:meUrl];

        MSURLSessionDataTask* meDataTask =
        [[MSURLSessionDataTask alloc]
            initWithRequest:meRequest
            client:self.graphClient
            completion:^(NSData *data, NSURLResponse *response, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                    return;
                }

                // Deserialize the response as a user
                NSError* graphError;
                MSGraphUser* user = [[MSGraphUser alloc] initWithData:data error:&graphError];

                if (graphError) {
                    completionBlock(nil, graphError);
                } else {
                    completionBlock(user, nil);
                }
            }];

        // Execute the request
        [meDataTask execute];
    }

    @end
    ```

1. <span data-ttu-id="ff7dc-143">Откройте **велкомевиевконтроллер. m** и добавьте приведенные `#import` ниже операторы в начало файла.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-143">Open **WelcomeViewController.m** and add the following `#import` statements at the top of the file.</span></span>

    ```objc
    #import "SpinnerViewController.h"
    #import "GraphManager.h"
    #import <MSGraphClientModels/MSGraphClientModels.h>
    ```

1. <span data-ttu-id="ff7dc-144">Добавьте в объявление `WelcomeViewController` интерфейса следующее свойство:</span><span class="sxs-lookup"><span data-stu-id="ff7dc-144">Add the following property to the `WelcomeViewController` interface declaration.</span></span>

    ```objc
    @property SpinnerViewController* spinner;
    ```

1. <span data-ttu-id="ff7dc-145">Замените существующий `viewDidLoad` код на следующий.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-145">Replace the existing `viewDidLoad` with the following code.</span></span>

    ```objc
    - (void)viewDidLoad {
        [super viewDidLoad];
        // Do any additional setup after loading the view.

        self.spinner = [SpinnerViewController alloc];
        [self.spinner startWithContainer:self];

        self.userProfilePhoto.image = [UIImage imageNamed:@"DefaultUserPhoto"];

        [GraphManager.instance
         getMeWithCompletionBlock:^(MSGraphUser * _Nullable user, NSError * _Nullable error) {
            dispatch_async(dispatch_get_main_queue(), ^{
                [self.spinner stop];

                if (error) {
                    // Show the error
                    UIAlertController* alert = [UIAlertController
                                                alertControllerWithTitle:@"Error getting user profile"
                                                message:error.debugDescription
                                                preferredStyle:UIAlertControllerStyleAlert];

                    UIAlertAction* okButton = [UIAlertAction
                                               actionWithTitle:@"OK"
                                               style:UIAlertActionStyleDefault
                                               handler:nil];

                    [alert addAction:okButton];
                    [self presentViewController:alert animated:true completion:nil];
                    return;
                }

                // Set display name
                self.userDisplayName.text = user.displayName ? : @"Mysterious Stranger";
                [self.userDisplayName sizeToFit];

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = user.mail ? : user.userPrincipalName;
                [self.userEmail sizeToFit];
            });
         }];
    }
    ```

<span data-ttu-id="ff7dc-146">Если вы сохраните изменения и перезапустите приложение, после того как вы обновите пользовательский интерфейс, отобразите отображаемое имя и адрес электронной почты пользователя.</span><span class="sxs-lookup"><span data-stu-id="ff7dc-146">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
