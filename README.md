# Login

This sample project shows how to present a login dialog using the Lock widget interface. Once you log in, you're taken to a very basic profile screen, with some data about your user.

##### Required Dependencies

```objective-c
  pod 'Lock', '~> 1.28'
  pod 'Lock/Safari'
```

##### Perform Social Authentication with SafariAuthenticator

First, go to your Client Dashboard and make sure that Allowed Callback URLs contains the following:

```
{YOUR_APP_BUNDLE_IDENTIFIER}://{YOUR_AUTH0_DOMAIN}/ios/{YOUR_APP_BUNDLE_IDENTIFIER}/callback
``` 
In your application's Info.plist file, register your iOS Bundle Identifier as a custom scheme. To do so, open the Info.plist as source code, and add this chunk of code under the main <dict> entry:

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>None</string>
        <key>CFBundleURLName</key>
        <string>auth0</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>{YOUR_APP_BUNDLE_IDENTIFIER}</string>
        </array>
    </dict>
</array>
```

#### Important Snippets

##### 1. Initialise Safari Authenticator in AppDelegate

In `AppDelegate.m`

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [A0LockLogger logAll];
    BOOL useUniversalLink = NO;
    A0Lock *lock = [A0Lock sharedLock];
    A0SafariAuthenticator *safari = [[A0SafariAuthenticator alloc] initWithLock:lock connectionName:@"google-oauth2" useUniversalLink:useUniversalLink];
    [lock registerAuthenticators:@[safari]];
    [lock applicationLaunchedWithOptions:launchOptions];
    return YES;
}
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    return [[A0Lock sharedLock] handleURL:url sourceApplication:sourceApplication];
}

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler {
    A0Lock *lock = [A0Lock sharedLock];
    return [lock continueUserActivity:userActivity restorationHandler:restorationHandler];
}
```

##### 2. Present the login widget

In `HomeViewController.m`:

```objective-c
- (IBAction)showLoginController:(id)sender {
    A0Lock *lock = [A0Lock sharedLock];
    
    A0LockViewController *controller = [lock newLockViewController];
    controller.onAuthenticationBlock = ^(A0UserProfile *profile, A0Token *token) {
        [self dismissViewControllerAnimated:YES completion:nil];
        [self performSegueWithIdentifier:@"ShowProfile" sender:profile];
    };
    
    [self presentViewController:controller animated:YES completion:nil];
}
```

##### 3. Pass the profile object

In this sample, the `profile` object is passed to the next screen through a variable, in the `prepareForSegue` method, as follows:

```objective-c
- (void) prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    if([segue.identifier isEqualToString:@"ShowProfile"]) {
        ProfileViewController *destViewController = segue.destinationViewController;
        destViewController.userProfile = sender;
    }
}
```

##### 4. Show basic profile data

In `ProfileViewController.m` we update the text label with the user's name, and load the profile image into the image view:

```objective-c
- (void) viewDidLoad {
    [super viewDidLoad];
    
    self.navigationItem.hidesBackButton = YES;
    self.welcomeLabel.text = [NSString stringWithFormat:@"Welcome, %@", self.userProfile.name];
    
    [[[NSURLSession sharedSession] dataTaskWithURL:self.userProfile.picture completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        dispatch_async(dispatch_get_main_queue(), ^{
            self.avatarImageView.image = [UIImage imageWithData:data];
        });
    }] resume];
}
```

