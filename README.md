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

## [Taking basic order details](https://www.hackingwithswift.com/books/ios-swiftui/taking-basic-order-details)

<img width="300" alt="スクリーンショット 2023-03-24 21 40 43" src="https://user-images.githubusercontent.com/47273077/227523878-8b3e28da-cafc-4e0d-bf13-645671e04855.png">

Order.swift
```swift
class Order: ObservableObject {
    static let types = ["Vanilla", "Strawberry", "Chocolate", "Rainbow"]
    
    @Published var type = 0
    @Published var quantity = 3
    
    @Published var specialRequestEnabled = false {
        didSet {
            if specialRequestEnabled == false {
                extraFrosting = false
                addSprinkles = false
            }
        }
    }
    @Published var extraFrosting = false
    @Published var addSprinkles = false
}
```

ContentView.swift
```swift
struct ContentView: View {
    @StateObject var order = Order()
    
    var body: some View {
        NavigationView {
            Form {
                Section {
                    Picker("Select your cake type", selection: $order.type) {
                        ForEach(Order.types.indices) {
                            Text(Order.types[$0])
                        }
                    }
                    
                    Stepper("Number of cakes \(order.quantity)", value: $order.quantity, in: 3...20)
                }
                
                Section {
                    Toggle("Any special requests?", isOn: $order.specialRequestEnabled.animation())
                    
                    if order.specialRequestEnabled {
                        Toggle("Add extra frosting", isOn: $order.extraFrosting)
                        Toggle("Add extra springkles", isOn: $order.addSprinkles)
                    }
                }
                
                Section {
                    NavigationLink {
                        AddressView(order: order)
                    } label: {
                        Text("Delivery details")
                    }
                }
            }
            .navigationTitle("Cupcake Corner")
        }
    }
}
```

## [Checking for a valid address](https://www.hackingwithswift.com/books/ios-swiftui/checking-for-a-valid-address)

<img width="300" alt="スクリーンショット 2023-03-25 6 56 46" src="https://user-images.githubusercontent.com/47273077/227651766-82cf4f35-5a24-4cbe-9fe9-5c409a3069bd.gif">

```swift
struct AddressView: View {
    @ObservedObject var order: Order
    
    var body: some View {
        Form {
            Section {
                TextField("Name", text: $order.name)
                TextField("Street address", text: $order.streetAddress)
                TextField("City", text: $order.city)
                TextField("Zip", text: $order.zip)
            }
            
            Section {
                NavigationLink {
                    CheckoutView(order: order)
                } label: {
                    Text("Check out")
                }
            }
            .disabled(order.hasValidAddress == false)
        }
        .navigationTitle("Delivey details")
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct AddressView_Previews: PreviewProvider {
    static var previews: some View {
        NavigationView {
            AddressView(order: Order())
        }
    }
}

```

Order.swift
```swift
class Order: ObservableObject {
    static let types = ["Vanilla", "Strawberry", "Chocolate", "Rainbow"]
    
    @Published var type = 0
    @Published var quantity = 3
    
    @Published var specialRequestEnabled = false {
        didSet {
            if specialRequestEnabled == false {
                extraFrosting = false
                addSprinkles = false
            }
        }
    }
    @Published var extraFrosting = false
    @Published var addSprinkles = false
    
    @Published var name = ""
    @Published var streetAddress = ""
    @Published var city = ""
    @Published var zip = ""
    
    var hasValidAddress: Bool {
        if name.isEmpty || streetAddress.isEmpty || city.isEmpty || zip.isEmpty {
            return false
        }
        
        return true
    }
}
```

## [Preparing for checkout](https://www.hackingwithswift.com/books/ios-swiftui/preparing-for-checkout)
<img width="300" alt="スクリーンショット 2023-03-25 6 56 46" src="https://user-images.githubusercontent.com/47273077/227655405-84cf615a-4cb9-4bdc-b634-1c3968f7fc57.gif">

CheckoutView.swift
```swift
struct CheckoutView: View {
    @ObservedObject var order: Order
    
    var body: some View {
        ScrollView {
            VStack {
                AsyncImage(url: URL(string: "https://hws.dev/img/cupcakes@3x.jpg"), scale: 3) { image in
                    image
                        .resizable()
                        .scaledToFit()
                } placeholder: {
                    ProgressView()
                }
                .frame(height: 233)
                
                Text("Your total is \(order.cost, format: .currency(code: "USD"))")
                    .font(.title)
                
                Button("Place Order", action: {})
                    .padding()
            }
        }
        .navigationTitle("Check out")
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct CheckoutView_Previews: PreviewProvider {
    static var previews: some View {
        NavigationView {
            CheckoutView(order: Order())
        }
    }
}
```

Order.swift
```swift
 var cost: Double {
        var cost = Double(quantity) * 2
        
        cost += (Double(type) / 2)
        
        if extraFrosting {
            cost += Double(quantity)
        }
        
        if addSprinkles {
            cost += Double(quantity) / 2
        }
        
        return cost
    }
```


