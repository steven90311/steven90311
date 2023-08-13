# 👋 Hi there, I’m Steven

## I’m an iOS Software Engineer.

## 😻 I love:

  - Coding
  - Designing
  - Playing Table Tennis and Guitar
  - Learning new about IT

## 🛠 Tools that I use:

  - Xcode
  - SourceTree
  - Postman
  - Figma
  - Sketch
  - AdobeXD
  - Jira
  - Confluence
  - GitHub
  - Bitbucket

## 🧰 Languages and Frameworks that I work with:

  - Swift
  - Clean Swift/MVVM
  - UIKit
  - AVFoundation
  - CoreLocation
  - Alamofire/Moya
  - Realm/CoreData
  - Apple Maps/Google Maps
  - Firebase
  - Google Places
  - Facebook SDK
  - OneSignal
  - Security
  - LocalAuthentication
  - StoreKit

## 💻 This is how I code:

#### Setting up App's global behavior 

```swift
class ApplicationGeneralSetupManager {
    
    static var shared = ApplicationGeneralSetupManager()
    
    func performGeneralSetup(_ launchOptions: [UIApplication.LaunchOptionsKey: Any]?) {
        setupAppDependencies(launchOptions)
        performRealmSetupAndMigration()
        setupGlobalAppearance()
        performOtherConfiguration()
    }
    
    //MARK: - Dependencies setup
    
    private func setupAppDependencies(_ launchOptions: [UIApplication.LaunchOptionsKey: Any]?) {
        /// Some setup code here 
    }
    
    // MARK: - Other setup
    
    private func performOtherConfiguration() {
        /// Some setup code here 
    }
    
    // MARK: - Realm setup
    
    private func performRealmSetupAndMigration() {
        /// Some setup code here 
    }
    
    // MARK: - Global appearance setup
    
    private func setupGlobalAppearance() {
        /// Some setup code here 
    }
}
```

#### Building UI Elements

```swift
    // MARK: - Object lifecycle
    
    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        setup()
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setup()
    }

    // MARK: - View lifecycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        setupProductsCollectionView()
        interactor?.prepareCollectionViewData()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        setupNotificationsButton()
    }

    // MARK: - Actions

    @IBAction func didPressNotifications(_ sender: UIBarButtonItem) {
        performSegue(withIdentifier: "NotificationsFromMenu", sender: self)
    }
    
    @objc
    private func didPullToRefresh(_ sender: Any) {
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.25) {
            self.interactor?.getAllProducts()
        }
    }
    
    func showPopupErrorMessage(title: String, body: String) {
        refreshControl.endRefreshing()
        showTopErrorView(title: title, message: body)
    }
    
    private func selectFirstFiltersCell() {
        let firstCellIndex = IndexPath(item: 0, section: 0)
        filtersCollectionView.selectItem(at: firstCellIndex, animated: false, scrollPosition: .centeredHorizontally)
    }
    
    // MARK: - UI Setup
    
    private func setupProductsCollectionView() {
        let columnLayout = ColumnFlowLayout(cellsPerRow: 2,
                                            minimumInterItemSpacing: 10,
                                            minimumLineSpacing: 10,
                                            sectionInset: UIEdgeInsets(top: 0, left: 12, bottom: 20, right: 12)
        )
        refreshControl.addTarget(self, action: #selector(didPullToRefresh(_:)), for: .valueChanged)
        
        productsCollectionView.alwaysBounceVertical = true
        productsCollectionView.refreshControl = refreshControl
        productsCollectionView.dataSource = self
        productsCollectionView.delegate = self
        productsCollectionView.collectionViewLayout = columnLayout
        productsCollectionView.contentInsetAdjustmentBehavior = .always
        
        productsCollectionView.register(UINib(nibName: "ProductCollectionViewCell", bundle: nil),
                                        forCellWithReuseIdentifier: "ProductCollectionViewCell")
    }
    
    private func setupNotificationsButton() {
        if GlobalData.enteredAsAGuest {
            notificationsBarButton.image = UIImage()
            return
        }
        if GlobalData.hasUnreadNotifications {
            notificationsBarButton.image = UIImage(named: "NotificationsFilledIcon")
        } else {
            notificationsBarButton.image = UIImage(named: "NotificationsIcon")
        }
    }
    
    // MARK: - Displaying Data
    
    func displayProducts(_ data: [ProductModel]) {
        refreshControl.endRefreshing()
        
        products = data
        
        productsCollectionView.reloadData()
    }
    
    func updateProducts() {
        interactor?.updateProductList()
    }
    
    // MARK: - Routing
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let scene = segue.identifier {
            let selector = NSSelectorFromString("routeTo\(scene)WithSegue:")
            if let router = router, router.responds(to: selector) {
                router.perform(selector, with: segue)
            }
        }
    }
```

#### Working with API's

```swift

    func performSignUpRequest(with data: SignUp.RequestModel) {
        print("Sign up request, \nPOST - \(RequestLinks.signUp)")
        
        AF.request(RequestLinks.signUp, method: .post, parameters: data, encoder: JSONParameterEncoder.default, headers: nil)
            .validate()
            .responseDecodable(of: SignUp.RegistrationResponse.self) { response in
                
                switch response.result {
                case .success(let responseData):
                    self.presenter?.openPhoneVerificationScene()
                case .failure(let error):
                    let errorMessage = APIErrorsHandler.handleErrorResponse(response: response.response,
                                                                            responseData: response.data,
                                                                            error: error)
                    
                    self.presenter?.showPopupErrorMessage(title: "Error",
                                                          body: errorMessage)
                }
            }
    }

    func getUserData(completion: @escaping (Result<UserModel, APIErrors>) -> Void) {
        let headers = APIManagerSetup.defaultHeader
        var requestLink = RequestLinks.userData
        
        if TokenManager.getUserId().isEmpty {
            completion(.error(.invalidUserId))
            return
        }
        
        requestLink = requestLink + "?userId=\(TokenManager.getUserId())"
        
        print("Get user data request, \nGET - ", requestLink)
        
        AF.request(requestLink, method: .get, parameters: nil, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseDecodable(of: UserModel.self) { response in
                
                switch response.result {
                case .success(let responseData):
                    TokenManager.save(token: responseData.accessToken, userId: responseData.userId)
                    completion(.success(responseData))
                case .failure(let error):
                    print("Alamofire error - ", error)
                    completion(.error(.unableToComplete))
                }
            }
    }

    func upload(photos: [UIImage], recognizedImageIds: [String], objectId: Int, objectType: Int, operationType: ItemDetails.ItemLibraryActions) {
        let endUrl = RequestLinks.uploadPhoto + "/\(objectType)/\(objectId)"
        let headers = APIManagerSetup.defaultHeader
        
        AF.upload(multipartFormData: { multipartFormData in
            
            for (index, photo) in photos.enumerated() {
                let imageData = photo.jpegData(compressionQuality: 0.25)
                
                for id in recognizedImageIds {
                    if let idData = id.data(using: .utf8) {
                        multipartFormData.append(idData, withName: "recognized_image_ids[]")
                    }
                }
                multipartFormData.append(imageData!, withName: "photos[\(index)]", fileName: "Image-\(index).png", mimeType: "image/png")
            }
        },
                  to: endUrl, method: .post , headers: headers)
            .uploadProgress(queue: .main, closure: { progress in
                print("Upload Progress: \(progress.fractionCompleted)")
            })
            .response { response in
                
                switch response.result {
                case .success(_):
                    print("Uploaded photo successfully.")
                    self.showPopupMessageBasedOn(operationType: operationType,
                                                 errorsWithPhotos: false)
                case .failure(let error):
                    print("Got error when uploading image.", error.localizedDescription)
                    self.showPopupMessageBasedOn(operationType: operationType,
                                                 errorsWithPhotos: true)
                }
            }
    }
```

#### Processing and storing data

```swift

    func updateOrderFromOrderDetails() {
        guard let selectedOrder = selectedOrder else { return }
        guard let currentDisplayingType = currentDisplayingType else { return }
        
        if let orderToModify = allOrders.enumerated().first(where: {$0.element.id == selectedOrder.id}) {
            allOrders[orderToModify.offset] = selectedOrder
        }
        
        self.currentOrders = allOrders.filter { $0.status != .cancelled && $0.status != .completed }
        self.pastOrders = allOrders.filter { $0.status != .preparing && $0.status != .ready }
        
        prepareAndDisplayOrders(of: currentDisplayingType)
    }
    
    func loadImageFromDocumentsDirectory(fileName: String) -> UIImage {
        let placeHolderImage = UIImage(named: "PlacesPlaceholder")!
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        let filePath = documentsURL.appendingPathComponent(fileName)
        
        let image = UIImage(contentsOfFile: filePath.path)
        if let savedImage = image {
            return savedImage
        } else {
            return placeHolderImage
        }
    }
    
    private func removeSavedFile(fileName: String) {
        let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
        let filePath = documentsURL.appendingPathComponent(fileName)
        do {
            if fileName.count > 0 {
                try FileManager.default.removeItem(at: filePath)
            }
        } catch let error as NSError {
            print("Error: \(error.domain)", error)
        }
    }

```


## 🗂 This is the list of my projects:


### [Scannur](https://apps.apple.com/us/app/scannur/id1560168842)

#### Barcode & image scanner software that helps companies to keep track of their products

##### Position:

  - Team Lead & Creator

###### Key Featues:

       - Barcode scanner;
       - Custom camera picker with image recognition;
       - Offline mode;
       - Multiple customizable roles;
       - Customizable item and list cards;
       - In-App purchases;

###### Tech Stack:

      - Swift
      - Clean Swift architecture
      - UIKit
      - AVFoundation
      - CoreLocation
      - Alamofire
      - Realm
      - Firebase Analytics
      - StoreKit



### [Urbs](https://apps.apple.com/us/app/urbs-smart-city-guides/id1499710519)

#### Travel guide app that allows users to explore different European cities

##### Position:

  - Team Lead & Creator

###### Key Featues:

       - Explore and listen to audio guides of multiple places;
       - Get dirrections to selected places;
       - Walk curated routes or build your own;
       - Download city data for offline usage;
       - Save your current accommodation in selected city;
       - In-App purchases;

###### Tech Stack:

      - Swift
      - Clean Swift architecture
      - UIKit
      - AVFoundation
      - CoreLocation
      - Google Maps
      - Alamofire
      - Realm
      - Firebase
      - StoreKit
      - Social Media Sign up & Log In



### [Do Happy](https://apps.apple.com/us/app/do-happy-daily-happier-habits/id1540858137)

#### Daily self care habit tracker

##### Position:

  - Team Lead

###### Key Featues:

       - Habit tracker;
       - Events calendar;
       - Important people organizer;
       - Daily challenges;
       - Social media sharing;
       - Customizable weekly task schedule;
       
###### Tech Stack:

      - Swift
      - MVC architecture
      - UIKit
      - AVFoundation
      - Moya 
      - Realm
      - Google Maps
      - Keychain
      - SnapKit
      - NaturalLanguage
      - Firebase Analytics
      - Facebook Analytics
      - Social Media Sign up & Log In



### [Crumb](https://apps.apple.com/us/app/crumb-fitness-belohnungen/id1455844646?platform=iphone)

#### Fitness tracker app where you get rewards for being active

##### Position:

  - Team Lead & Creator

###### Key Featues:

       - Track steps, floors and distance covered every day;
       - Complete everyday challenges based on your activity;
       - Receive progressive rewards;
       - Purchase discounts by rewards earned for activity;
       - Invite friends and get bonuses;

###### Tech Stack:

      - Swift
      - Viper architecture
      - UIKit
      - Alamofire
      - Firebase 
      - Firebase Analytics
      - Health Kit


### [RoomSnap](https://apps.apple.com/us/app/roomsnap/id1526278342)

#### Real Estate photograpgher & editor

##### Position:

  - Team Lead

###### Key Featues:

       - Create your properties by taking pictures of them;
       - Get access to professional photo editors;
       - Receive professionaly eddited images;

###### Tech Stack:

      - Swift
      - MVVM architecture
      - UIKit
      - AVFoundation
      - Moya 
      - Firebase Analytics
      - Keychain
      - Audio Toolbox
