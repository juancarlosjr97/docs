<amplify-block-switcher>

<amplify-block name="Listener (iOS 11+)">

```swift
func signIn(username: String, password: String) {
    _ = Amplify.Auth.signIn(username: username, password: password) { result in
        switch result {
        case .success(_):
            print("Sign in succeeded")
        case .failure(let error):
            print("Sign in failed \(error)")
        }
    }
}
```

</amplify-block>

<amplify-block name="Combine (iOS 13+)">

```swift
func signIn(username: String, password: String) -> AnyCancellable {
    Amplify.Auth.signIn(username: username, password: password)
        .resultPublisher
        .sink {
            if case let .failure(authError) = $0 {
                print("Fetch session failed with error \(authError)")
            }
        }
        receiveValue: { _ in
            print("Sign in succeeded")
        }
}
```

</amplify-block>

</amplify-block-switcher>