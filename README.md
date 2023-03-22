# CupcakeCorner
## [Adding Codable conformance for @Published properties](https://www.hackingwithswift.com/books/ios-swiftui/adding-codable-conformance-for-published-properties)

```swift
class User: ObservableObject, Codable {
    enum CodingKeys: CodingKey {
        case name
    }
    
    @Published var name = "Paul Hudson"
    
    required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        name = try container.decode(String.self, forKey: .name)
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(name, forKey: .name)
    }
}
```

## [Codable data with URLSession and SwiftUI](https://www.hackingwithswift.com/books/ios-swiftui/sending-and-receiving-codable-data-with-urlsession-and-swiftui)
<img width="300" alt="スクリーンショット 2023-03-22 16 47 31" src="https://user-images.githubusercontent.com/47273077/226834631-0cf9eb6f-ef2e-4e84-8402-497e330b1f61.png">

```swift
import SwiftUI

struct Response: Codable {
    var results: [Result]
}

struct Result: Codable {
    var trackId: Int
    var trackName: String
    var collectionName: String
}

struct ContentView: View {
    @State private var results = [Result]()
    
    var body: some View {
        List(results, id: \.trackId) { item in
            VStack(alignment: .leading) {
                Text(item.trackName)
                    .font(.headline)
                Text(item.collectionName)
            }
        }
        .task {
            await loadData()
        }
    }
    
    func loadData() async {
        guard let url = URL(string: "https://itunes.apple.com/search?term=taylor+swift&entity=song") else {
            print("Invalid URL")
            return
        }
        
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            
            if let decodedResponse = try? JSONDecoder().decode(Response.self, from: data) {
                results = decodedResponse.results
            }
        } catch {
            print("Invalid data")
        }
    }
}
```

### [Loading an image from a remote server](https://www.hackingwithswift.com/books/ios-swiftui/loading-an-image-from-a-remote-server)

<img width="300" alt="スクリーンショット 2023-03-22 16 47 31" src="https://user-images.githubusercontent.com/47273077/226838521-7643a7e6-1279-4325-bb2a-a5c9c9b67ef0.gif">

```swift
struct ContentView: View {
    
    var body: some View {
        AsyncImage(url: URL(string: "http://hws.dev/img/logo.png")) { image in
            image
                .resizable()
                .scaledToFit()
        } placeholder: {
            ProgressView()
        }
        .frame(width: 200, height: 200)
            
    }
```

<img width="300" alt="スクリーンショット 2023-03-22 16 47 31" src="https://user-images.githubusercontent.com/47273077/226840141-a7037d30-be1c-4292-b6e9-bf268e04f269.gif">

```swift
struct ContentView: View {
    
    var body: some View {
        AsyncImage(url: URL(string: "https://hws.dev/img/bad.png")) { phase in
            if let image = phase.image {
                image
                    .resizable()
                    .scaledToFit()
            } else if phase.error != nil {
                Text("There was an error loading the image")
            } else {
                ProgressView()
            }
        }
        .frame(width: 200, height: 200)
    }
     
}
```

### [Validating and disabling forms](https://www.hackingwithswift.com/books/ios-swiftui/validating-and-disabling-forms)

<img width="300" alt="スクリーンショット 2023-03-22 17 39 50" src="https://user-images.githubusercontent.com/47273077/226846928-33b63c77-3f13-4688-af57-1881acbe6149.png">

```swift
struct ContentView: View {
    @State private var username = ""
    @State private var email = ""
    
    var body: some View {
        Form {
            Section {
                TextField("username", text: $username)
                TextField("Email", text: $email)
            }
            
            Section {
                Button("Create account") {
                    print("Creating account_")
                }
            }
            .disabled(disableForm)
        }
    }

    var disableForm: Bool {
        username.count < 5 || email.count < 5
    }
}
```
