## Hi there 👋

<!--
**ennistudio/ENNIstudio** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
import SwiftUI
import PencilKit

// MARK: - Main App
@main
struct ENNIStudioApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// MARK: - Content View
struct ContentView: View {
    @State private var canvasView = PKCanvasView()
    @State private var toolPicker = PKToolPicker()
    
    // Brush settings
    @State private var brushType: PKInkingTool.InkType = .pen
    @State private var brushColor: UIColor = .black
    @State private var brushSize: CGFloat = 5.0
    
    // Language toggle: true = English, false = Georgian
    @State private var englishMode = false
    
    // UI toggles
    @State private var showingColorPicker = false
    @State private var showingBrushMenu = false
    
    // Layers
    @State private var layers: [PKDrawing] = [PKDrawing()]
    @State private var currentLayerIndex = 0
    
    var body: some View {
        VStack(spacing: 0) {
            
            // Top Toolbar
            HStack(spacing: 15) {
                
                Button(action: undo) { Image(systemName: "arrow.uturn.left") }
                    .accessibilityLabel(englishMode ? "Undo" : "უკან")
                
                Button(action: redo) { Image(systemName: "arrow.uturn.right") }
                    .accessibilityLabel(englishMode ? "Redo" : "წინა")
                
                Button(action: { showingColorPicker.toggle() }) {
                    Circle().fill(Color(brushColor)).frame(width: 30, height: 30)
                }
                
                Button(action: { showingBrushMenu.toggle() }) {
                    Image(systemName: "paintbrush").font(.title2)
                        .accessibilityLabel(englishMode ? "Brushes" : "ფუნჯები")
                }
                
                Slider(value: $brushSize, in: 1...50) {
                    Text(englishMode ? "Brush Size" : "ფუნჯის ზომა")
                }
                
                Button(action: exportImage) {
                    Image(systemName: "square.and.arrow.up")
                        .accessibilityLabel(englishMode ? "Export" : "ექსპორტი")
                }
                
                // Language switch
                Toggle(englishMode ? "EN" : "GE", isOn: $englishMode)
                    .toggleStyle(SwitchToggleStyle(tint: .blue))
            }
            .padding()
            .background(Color(.systemGray6))
            
            // Canvas
            CanvasViewWrapper(
                canvasView: $canvasView,
                toolPicker: $toolPicker,
                brushType: $brushType,
                brushColor: $brushColor,
                brushSize: $brushSize,
                layers: $layers,
                currentLayerIndex: $currentLayerIndex
            )
            .edgesIgnoringSafeArea(.all)
        }
        .sheet(isPresented: $showingColorPicker) {
            ColorPicker(englishMode ? "Pick Color" : "ფერი აირჩიე", selection: Binding(get: {
                Color(brushColor)
            }, set: { newColor in
                brushColor = UIColor(newColor)
            }))
            .padding()
        }
        .actionSheet(isPresented: $showingBrushMenu) {
            ActionSheet(title: Text(englishMode ? "Brushes" : "ფუნჯები"), buttons: [
                .default(Text(englishMode ? "Pencil" : "სანათი")) { brushType = .pencil },
                .default(Text(englishMode ? "Ink" : "მელანი")) { brushType = .pen },
                .default(Text(englishMode ? "Oil Paint" : "ზეთი")) { brushType = .marker },
                .default(Text(englishMode ? "Watercolor" : "წყალი")) { brushType = .pen },
                .default(Text(englishMode ? "Charcoal" : "ანთება")) { brushType = .pencil },
                .default(Text(englishMode ? "Marker" : "მარკერი")) { brushType = .marker },
                .default(Text(englishMode ? "Calligraphy" : "კალიგრაფია")) { brushType = .pen },
                .default(Text(englishMode ? "Airbrush" : "აეროზოლი")) { brushType = .marker },
                .cancel()
            ])
        }
    }
    
    // MARK: - Functions
    func undo() { canvasView.undoManager?.undo() }
    func redo() { canvasView.undoManager?.redo() }
    
    func exportImage() {
        let image = canvasView.drawing.image(from: canvasView.bounds, scale: UIScreen.main.scale)
        UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
        print(englishMode ? "Saved to Photos" : "შენახულია გალერეაში")
    }
}

// MARK: - Canvas Wrapper
struct CanvasViewWrapper: UIViewRepresentable {
    @Binding var canvasView: PKCanvasView
    @Binding var toolPicker: PKToolPicker
    @Binding var brushType: PKInkingTool.InkType
    @Binding var brushColor: UIColor
    @Binding var brushSize: CGFloat
    @Binding var layers: [PKDrawing]
    @Binding var currentLayerIndex: Int
    
    func makeUIView(context: Context) -> PKCanvasView {
        canvasView.drawingPolicy = .anyInput
        canvasView.backgroundColor = UIColor.systemBackground
        canvasView.allowsFingerDrawing = true
        
        if let window = UIApplication.shared.windows.first {
            toolPicker.setVisible(true, forFirstResponder: canvasView)
            toolPicker.addObserver(canvasView)
            canvasView.becomeFirstResponder()
        }
        updateBrush()
        return canvasView
    }
    
    func updateUIView(_ uiView: PKCanvasView, context: Context) {
        updateBrush()
    }
    
    func updateBrush() {
        let ink = PKInkingTool(brushType, color: brushColor, width: brushSize)
        canvasView.tool = ink
    }
}
