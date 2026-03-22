## Hi there 👋

<!--
**ennistudio/ENNIstudio** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
import SwiftUI
import PencilKit

// MARK: - Layer Management Functions
extension ContentView {
    
    func addLayer() {
        layers.append(PKDrawing())
        currentLayerIndex = layers.count - 1
        loadCurrentLayer()
    }
    
    func deleteCurrentLayer() {
        guard layers.count > 1 else { return } // მინიმუმ 1 უნდა იყოს
        layers.remove(at: currentLayerIndex)
        currentLayerIndex = max(0, currentLayerIndex - 1)
        loadCurrentLayer()
    }
    
    func duplicateCurrentLayer() {
        let newLayer = layers[currentLayerIndex]
        layers.insert(newLayer, at: currentLayerIndex + 1)
        currentLayerIndex += 1
        loadCurrentLayer()
    }
    
    func switchLayer(to index: Int) {
        guard layers.indices.contains(index) else { return }
        currentLayerIndex = index
        loadCurrentLayer()
    }
    
    func toggleLayerVisibility(index: Int) {
        // PencilKit-მა ოფიციალურად არ აქვს visibility, workaround: ზარის ქულას გავაკეთებთ ალფაზე ან ცალკე canvas overlay
        // მარტივი ვერსია: მხოლოდ სელექცია
    }
    
    func loadCurrentLayer() {
        canvasView.drawing = layers[currentLayerIndex]
    }
    
    func saveCurrentLayer() {
        layers[currentLayerIndex] = canvasView.drawing
    }
}

// MARK: - File Management
extension ContentView {
    
    func saveDrawing(to name: String) {
        saveCurrentLayer()
        let combinedDrawing = PKDrawing()
        layers.forEach { combinedDrawing.strokes.append(contentsOf: $0.strokes) }
        do {
            let data = try combinedDrawing.dataRepresentation()
            let url = getDocumentsDirectory().appendingPathComponent("\(name).drawing")
            try data.write(to: url)
            print("Saved to \(url)")
        } catch {
            print("Failed to save: \(error.localizedDescription)")
        }
    }
    
    func loadDrawing(from url: URL) {
        do {
            let data = try Data(contentsOf: url)
            let drawing = try PKDrawing(data: data)
            layers = [drawing]
            currentLayerIndex = 0
            loadCurrentLayer()
        } catch {
            print("Failed to load: \(error.localizedDescription)")
        }
    }
    
    private func getDocumentsDirectory() -> URL {
        FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    }
}

// MARK: - Brush Mapping Fix
extension ContentView {
    func updateBrush() {
        let toolType: PKInkingTool.InkType
        switch brushType {
        case .pencil:
            toolType = .pencil
        case .pen:
            toolType = .pen
        case .marker:
            toolType = .marker
        default:
            toolType = .pen
        }
        canvasView.tool = PKInkingTool(toolType, color: brushColor, width: brushSize)
    }
}

// MARK: - Zoom Controls
extension ContentView {
    func zoomCanvas(by scale: CGFloat) {
        canvasView.transform = CGAffineTransform(scaleX: scale, y: scale)
    }
}

// MARK: - New Toolbar Buttons (Example)
extension ContentView {
    var layerToolbar: some View {
        HStack(spacing: 10) {
            Button(action: addLayer) { Image(systemName: "plus.square.on.square") }
            Button(action: duplicateCurrentLayer) { Image(systemName: "doc.on.doc") }
            Button(action: deleteCurrentLayer) { Image(systemName: "trash") }
            
            Picker(selection: $currentLayerIndex, label: Text("")) {
                ForEach(0..<layers.count, id: \.self) { idx in
                    Text("Layer \(idx + 1)").tag(idx)
                }
            }
            .onChange(of: currentLayerIndex) { _ in
                loadCurrentLayer()
            }
        }
        .padding()
        .background(Color(.systemGray5))
    }
}

// MARK: - Settings Example
struct SettingsView: View {
    @Binding var brushSize: CGFloat
    @Binding var brushColor: UIColor
    @Binding var englishMode: Bool
    
    var body: some View {
        Form {
            Section(header: Text(englishMode ? "Brush" : "ფუნჯი")) {
                Slider(value: $brushSize, in: 1...50, label: { Text("Size") })
                ColorPicker(englishMode ? "Color" : "ფერი", selection: Binding(get: { Color(brushColor) }, set: { brushColor = UIColor($0) }))
            }
            Section(header: Text(englishMode ? "Language" : "ენა")) {
                Toggle("English Mode", isOn: $englishMode)
            }
        }
    }
}
    }
}
