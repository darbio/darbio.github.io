---
layout: post
title: "SwiftUI binding in UIKit with UIHostingController and ObservableObject"
date: 2023-05-07 00:00
comments: false
categories: swiftui swift uikit ios
---

Often I have found myself wanting to use SwiftUI data bindings in Views hosted via UIHostingController in UIKit. A great example of this is when you want to use a SwiftUI View in a MKAnnotationView which has a data binding in the SwiftUI View.

Obviously, UIKit and SwiftUI don't work with the `@State` of `@Binding` property wrappers, however `@ObservedObject` still works well and changes made on the UIKit code are propagated down to the SwiftUI View. Magic!

This allows us to create a View Model for our SwiftUI View, declare it on the UIKit class and bind the changes in the UIKit code to the SwiftUI View.

The SwiftUI code:

```
struct MapMarkerView: View {
    class ViewModel : ObservableObject {
        @Published var color: Color = .red
    }
    
    @ObservedObject var viewModel: ViewModel
 
    var body: some View {
        Rectangle()
          .fill(self.color)
    }
}
```

The UIKit code:

```
class MapMarkerAnnotationView : MKAnnotationView {
    var viewModel = MapMarkerView.ViewModel()
    
    override init(annotation: MKAnnotation?, reuseIdentifier: String?) {
        super.init(annotation: annotation, reuseIdentifier: reuseIdentifier)
        
        let vc = UIHostingController(rootView: MapMarkerView(viewModel: self.viewModel))
        guard let view = vc.view {
            view.frame = bounds
            addSubview(view)
        }
    }
    
    @available(*, unavailable)
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func setColor(color: UIColor) {
        self.viewModel.color = Color(color)
    }
}
```

Now when you call `setColor` the color in the `MapMarkerView` will be updated through the `ObservedObject` binding. Pretty neat!
