# RealityKit-ARQuickLook-SwiftUI
It is not easy to place .reality files in Swift playgroud. This program supports gestures while ensuring that the .reality/usdz file is fully opened.
# The first part
![IMG_1111](https://github.com/S-way520/RealityKit-ARQuickLook-SwiftUI/assets/95877651/a7e481ac-7741-40cd-8790-112dd2e3b78b)
# The second part
Code file 1
```swift
import SwiftUI
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

```
Code file 2
```swift
import SwiftUI
import RealityKit
import Combine
struct ContentView: View {
    var body: some View {
        let usdzURL = Bundle.main.url(forResource: "F8a11", withExtension: "reality")!
            ARViewContainer(usdzURL: usdzURL)
                .edgesIgnoringSafeArea(.all)
    }
}
struct ARViewContainer: UIViewRepresentable {
    let arView = ARView(frame: .zero)
    let usdzURL: URL
    func makeUIView(context: Context) -> ARView {
        arView.addGestureRecognizer(UIRotationGestureRecognizer(target: context.coordinator, action: #selector(Coordinator.rotated)))
        arView.addGestureRecognizer(UIPinchGestureRecognizer(target: context.coordinator, action: #selector(Coordinator.pinched)))
        arView.addGestureRecognizer(UIPanGestureRecognizer(target: context.coordinator, action: #selector(Coordinator.didPan(_:))))
        return arView
    }
    func updateUIView(_ uiView: ARView, context: Context) {
        updateCounter(uiView: uiView)
    }
    func makeCoordinator() -> Coordinator {
        return Coordinator(arViewContainer: self)
    }
    private func updateCounter(uiView: ARView) {
        uiView.scene.anchors.removeAll()
        let anchor = AnchorEntity(plane: .horizontal)
        let modelEntity = try! ModelEntity.load(contentsOf: usdzURL)
        modelEntity.setParent(anchor)
        uiView.scene.addAnchor(anchor)
    }
    class Coordinator {
        let arViewContainer: ARViewContainer
        init(arViewContainer: ARViewContainer) {
            self.arViewContainer = arViewContainer
        }
        func gestureRecognizer(_ gesture: UIGestureRecognizer) -> Bool {
            return true
        }
        
        @objc func rotated(_ gesture: UIRotationGestureRecognizer) {
            if let anchor = arViewContainer.arView.scene.anchors.first {
                var transform = anchor.transform
                transform.rotation *= simd_quatf(angle: Float(gesture.rotation), axis: [0, -1, 0])
                anchor.transform = transform
            }
            gesture.rotation = 0
        }
        @objc func pinched(_ gesture: UIPinchGestureRecognizer) {
            if let anchor = arViewContainer.arView.scene.anchors.first {
                var transform = anchor.transform
                let scale = Float(gesture.scale)
                transform.scale *= simd_float3(scale, scale, scale)
                anchor.transform = transform
            }
            gesture.scale = 1
        }
        @objc func didPan(_ gesture: UIPanGestureRecognizer) {
            if let camera = arViewContainer.arView.session.currentFrame?.camera {
                let cameraTransform = camera.transform//.translation
                let cameraPosition = SIMD3(cameraTransform.columns.3.x, cameraTransform.columns.3.y, cameraTransform.columns.3.z)
                
                let Hypotenuse = sqrt(cameraPosition.y * cameraPosition.y + cameraPosition.x * cameraPosition.x)
                
                let Sin = cameraPosition.y / Hypotenuse
                let Cos = cameraPosition.x / Hypotenuse
                
                if let anchor = arViewContainer.arView.scene.anchors.first {
                    var transform = anchor.transform
                    let translation = gesture.translation(in: arViewContainer.arView)
                    
                    let xDelta = Float(translation.x) / 1000
                    let yDelta = Float(translation.y) / 1000
                    
                    let rotatedX = xDelta * Sin + yDelta * Cos
                    let rotatedY = yDelta * Sin - xDelta * Cos
                    
                    let translationDelta = SIMD3( rotatedX, 0, rotatedY)
                    transform.translation += translationDelta
                    anchor.transform = transform
                }
                gesture.setTranslation(.zero, in: arViewContainer.arView)
            }
            
        }
        /*
        @objc func didPan(_ gesture: UIPanGestureRecognizer) {
            if let anchor = arViewContainer.arView.scene.anchors.first {
                var transform = anchor.transform
                let translation = gesture.translation(in: arViewContainer.arView)
                let translationDelta = SIMD3(Float(translation.x) / 1000, 0, Float(translation.y) / 1000)
                transform.translation += translationDelta
                anchor.transform = transform
            }
            gesture.setTranslation(.zero, in: arViewContainer.arView)
        }
         */
        
    }
}
```
