# SpriteKit Cheatsheet

## Sprite Nodes

*15 March 2024*

- SKTextureFilteringMode
- usesMipmaps

## Shape nodes

*14 March 2024*

> **TL;DR**: Nodes of type `SKShapeNode` can generate shapes with code, providing the ability to draw and edit shapes dynamically during runtime. SpriteKit's ability to generate accurate physics bodies for shape nodes is more limited than it is with sprite nodes. When rendered, a shape node is rasterized, so when it's drawn in a size other than its native size, some filtering is applied (blurring or aliasing).

Nodes of type `SKShapeNode` are generated procedurally. Their shape, color fill, and border style are defined with code. By default, a shape is transparent (it has no fill color) and has a one-point thick white border.

```swift
let circle = SKShapeNode(circleOfRadius: 50)
let ellipse1 = SKShapeNode(ellipseOf: CGSize(width: 50, height: 100))
let ellipse2 = SKShapeNode(ellipseIn: CGRect(x: 0, y: 0, width: 50, height: 50))
```

<img src="Screenshots/SpriteKit shape nodes - 1.png" alt="SpriteKit shape nodes - 1" style="width:25%;" />

Notice the code for the second ellipse. It is created by passing a `CGRect` data type instead of `CGSize`. This is an important distinction: CGRect is a data type that contains the width and height of the shape, as well as **how the path is positioned relative to the node's origin**. If we replace the last line above with this one:

```swift
let ellipse2 = SKShapeNode(ellipseIn: CGRect(x: -25, y: 25, width: 50, height: 50))
```

<img src="Screenshots/SpriteKit shape nodes - 2.png" alt="SpriteKit shape nodes - 2" style="width:25%;" />

The ellipse will be pushed 25 points to the left of the node's origin, and 25 points to the bottom of the node's origin. Since the width and height of the ellipse are both 50 points, that will make the ellipse centered around the node's origin!

CGRect is a common data type in SpriteKit. It provides control over the origin of a programmatic path relative to its parent's origin. When the ellipse is defined with CGSize instead of CGRect, it is automatically centered around the node's origin, which we can get with CGRect provided we do additional processing:

```swift
let length: CGFloat = 50 // a length for both width and height
let ellipseIn = SKShapeNode(ellipseIn: CGRect(x: -length/2, y: -length/2, width: length, height: length))
```

In SpriteKit, you can draw shapes using:

- Rectangles, with a width and a height, and with or without a corner radius
- Circles, with radius
- Ellipses, with width and height
- Path, from a rectangle or a circle
- Path, from arcs of a circle
- Path, with straight lines from a collection of points
- Path, with curved lines from a collection of points, where pairs of point are joined with a quadratic curve

A note about `SKShapeNode` rendering: even though shapes are procedural, like a vector shape would be, SpriteKit's renderer itself rasterize the shape during runtime. In fact, every node that draws is rasterized by SpriteKit's renderer, visually behaving like a sprite. Therefore, if you zoom in on shape node, if it its counters do not snap to the pixel grid, it will be either blurred or aliased, depending on the node's filtering mode. 

## Selection and retrieval

*14 March 2024*

```swift
// select a node by name, no matter how deep in the hierarchy
// the `//` means that the search doesn't stop to the immediate children nodes
parentNode.childNode(withName: "//nodeName")

// code for the first node matching the given name 
childNode(withName: "nodeName") {node, _ in
                                 
}

// code for every node with the given name
enumerateChildNodes(withName: "nodeName") {node, _ in
	
}

// same as above, with a different Swifty syntax
enumerateChildNodes(withName: "nodeName", using: { node, _ in
	
})

// code for every node with the given name
// Use `stop.pointee = true` or `stop[0] = true` to stop the enumeration
enumerateChildNodes(withName: "nodeName") {node, stop in
	
}

// loop through all the children of the node
for child in children {
    
}

// code for myNode with name "targetName"
if let myNode = self.childNode(withName: "//targetName") {
    
}
```

## Manipulating the scene graph

*14 March 2024*

```swift
// switch the parent of a node.
// See https://developer.apple.com/documentation/spritekit/sknode/accessing_and_modifying_the_node_tree
nodeToMove.move(toParent: self)
```

## Styling text

*13 March 2024*

In addition to `fontName`, `fontSize`, and `fontColor`, there are other text properties that SpriteKit provides out of the box, such as:

```swift
// a value of 1 will constrain text to one line
// a value of 0 will wrap text over as many lines as needed
myLabel.numberOfLines = 0
// the width, in screen points, after which line-break mode should be applied
// the default is 0, which means that no line-break is applied
myLabel.preferredMaxLayoutWidth = 360
// determines the line-break mode for multiple lines
myLabel.lineBreakMode = .byTruncatingTail
```

The last property, `lineBreakMode`, inherits from TextKit. This hints to other hidden possibilities with which we can style text in SpriteKit. Indeed, a `SKLabelNode` has a property called `attributedText`, which is a bridge to `NSAttributedString`, a large class with many properties and settings to control and style text. We can borrow some of these features and apply them to a SpriteKit label node. For example:

<img src="Screenshots/SpriteKit-attributedText-examples.png" alt="SpriteKit-attributedText-examples" style="width:50%;" />

```swift
let text = "BAM!"

let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.alignment = .center
paragraphStyle.lineHeightMultiple = 1

let shadow = NSShadow()
shadow.shadowOffset = CGSize(width: 0, height: 10)
shadow.shadowColor = SKColor.black.withAlphaComponent(0.3)
shadow.shadowBlurRadius = 20

let attributes: [NSAttributedString.Key: Any] = [
    .paragraphStyle: paragraphStyle,
    .font: UIFont(name: "ChalkboardSE-Bold", size: 100)!,
    .foregroundColor: SKColor(red: 1, green: 0.95, blue: 0, alpha: 1),
    .strokeColor: SKColor(red: 1, green: 0, blue: 0.38, alpha: 1),
    .strokeWidth: -5,
    .shadow: shadow
]

let label = SKLabelNode()
label.attributedText = NSAttributedString(string: text, attributes: attributes)
label.numberOfLines = 0
label.verticalAlignmentMode = .center
addChild(label)
```

## Adding a font

*13 March 2024*

Out of the box, you can print the fonts that will work with SpriteKit with this code:

```swift
for family in UIFont.familyNames.sorted() {
    let names = UIFont.fontNames(forFamilyName: family)
    print("Family: \(family) Font names: \(names)")
}
```

Here is what I get on my Mac:

```swift
Family: Academy Engraved LET Font names: ["AcademyEngravedLetPlain"]
Family: Al Nile Font names: ["AlNile", "AlNile-Bold"]
Family: American Typewriter Font names: ["AmericanTypewriter", "AmericanTypewriter-Light", "AmericanTypewriter-Semibold", "AmericanTypewriter-Bold", "AmericanTypewriter-Condensed", "AmericanTypewriter-CondensedLight", "AmericanTypewriter-CondensedBold"]
Family: Apple Color Emoji Font names: ["AppleColorEmoji"]
Family: Apple SD Gothic Neo Font names: ["AppleSDGothicNeo-Regular", "AppleSDGothicNeo-Thin", "AppleSDGothicNeo-UltraLight", "AppleSDGothicNeo-Light", "AppleSDGothicNeo-Medium", "AppleSDGothicNeo-SemiBold", "AppleSDGothicNeo-Bold"]
Family: Apple Symbols Font names: ["AppleSymbols"]
Family: Arial Font names: ["ArialMT", "Arial-ItalicMT", "Arial-BoldMT", "Arial-BoldItalicMT"]
Family: Arial Hebrew Font names: ["ArialHebrew", "ArialHebrew-Light", "ArialHebrew-Bold"]
Family: Arial Rounded MT Bold Font names: ["ArialRoundedMTBold"]
Family: Avenir Font names: ["Avenir-Book", "Avenir-Roman", "Avenir-BookOblique", "Avenir-Oblique", "Avenir-Light", "Avenir-LightOblique", "Avenir-Medium", "Avenir-MediumOblique", "Avenir-Heavy", "Avenir-HeavyOblique", "Avenir-Black", "Avenir-BlackOblique"]
Family: Avenir Next Font names: ["AvenirNext-Regular", "AvenirNext-Italic", "AvenirNext-UltraLight", "AvenirNext-UltraLightItalic", "AvenirNext-Medium", "AvenirNext-MediumItalic", "AvenirNext-DemiBold", "AvenirNext-DemiBoldItalic", "AvenirNext-Bold", "AvenirNext-BoldItalic", "AvenirNext-Heavy", "AvenirNext-HeavyItalic"]
Family: Avenir Next Condensed Font names: ["AvenirNextCondensed-Regular", "AvenirNextCondensed-Italic", "AvenirNextCondensed-UltraLight", "AvenirNextCondensed-UltraLightItalic", "AvenirNextCondensed-Medium", "AvenirNextCondensed-MediumItalic", "AvenirNextCondensed-DemiBold", "AvenirNextCondensed-DemiBoldItalic", "AvenirNextCondensed-Bold", "AvenirNextCondensed-BoldItalic", "AvenirNextCondensed-Heavy", "AvenirNextCondensed-HeavyItalic"]
Family: Baskerville Font names: ["Baskerville", "Baskerville-Italic", "Baskerville-SemiBold", "Baskerville-SemiBoldItalic", "Baskerville-Bold", "Baskerville-BoldItalic"]
Family: Bodoni 72 Font names: ["BodoniSvtyTwoITCTT-Book", "BodoniSvtyTwoITCTT-BookIta", "BodoniSvtyTwoITCTT-Bold"]
Family: Bodoni 72 Oldstyle Font names: ["BodoniSvtyTwoOSITCTT-Book", "BodoniSvtyTwoOSITCTT-BookIt", "BodoniSvtyTwoOSITCTT-Bold"]
Family: Bodoni 72 Smallcaps Font names: ["BodoniSvtyTwoSCITCTT-Book"]
Family: Bodoni Ornaments Font names: ["BodoniOrnamentsITCTT"]
Family: Bradley Hand Font names: ["BradleyHandITCTT-Bold"]
Family: Chalkboard SE Font names: ["ChalkboardSE-Regular", "ChalkboardSE-Light", "ChalkboardSE-Bold"]
Family: Chalkduster Font names: ["Chalkduster"]
Family: Charter Font names: ["Charter-Roman", "Charter-Italic", "Charter-Bold", "Charter-BoldItalic", "Charter-Black", "Charter-BlackItalic"]
Family: Cochin Font names: ["Cochin", "Cochin-Italic", "Cochin-Bold", "Cochin-BoldItalic"]
Family: Copperplate Font names: ["Copperplate", "Copperplate-Light", "Copperplate-Bold"]
Family: Courier New Font names: ["CourierNewPSMT", "CourierNewPS-ItalicMT", "CourierNewPS-BoldMT", "CourierNewPS-BoldItalicMT"]
Family: DIN Alternate Font names: ["DINAlternate-Bold"]
Family: DIN Condensed Font names: ["DINCondensed-Bold"]
Family: Damascus Font names: ["Damascus", "DamascusLight", "DamascusMedium", "DamascusSemiBold", "DamascusBold"]
Family: Devanagari Sangam MN Font names: ["DevanagariSangamMN", "DevanagariSangamMN-Bold"]
Family: Didot Font names: ["Didot", "Didot-Italic", "Didot-Bold"]
Family: Euphemia UCAS Font names: ["EuphemiaUCAS", "EuphemiaUCAS-Italic", "EuphemiaUCAS-Bold"]
Family: Farah Font names: ["Farah"]
Family: Futura Font names: ["Futura-Medium", "Futura-MediumItalic", "Futura-Bold", "Futura-CondensedMedium", "Futura-CondensedExtraBold"]
Family: Galvji Font names: ["Galvji", "Galvji-Bold"]
Family: Geeza Pro Font names: ["GeezaPro", "GeezaPro-Bold"]
Family: Georgia Font names: ["Georgia", "Georgia-Italic", "Georgia-Bold", "Georgia-BoldItalic"]
Family: Gill Sans Font names: ["GillSans", "GillSans-Italic", "GillSans-Light", "GillSans-LightItalic", "GillSans-SemiBold", "GillSans-SemiBoldItalic", "GillSans-Bold", "GillSans-BoldItalic", "GillSans-UltraBold"]
Family: Grantha Sangam MN Font names: ["GranthaSangamMN-Regular", "GranthaSangamMN-Bold"]
Family: Helvetica Font names: ["Helvetica", "Helvetica-Oblique", "Helvetica-Light", "Helvetica-LightOblique", "Helvetica-Bold", "Helvetica-BoldOblique"]
Family: Helvetica Neue Font names: ["HelveticaNeue", "HelveticaNeue-Italic", "HelveticaNeue-UltraLight", "HelveticaNeue-UltraLightItalic", "HelveticaNeue-Thin", "HelveticaNeue-ThinItalic", "HelveticaNeue-Light", "HelveticaNeue-LightItalic", "HelveticaNeue-Medium", "HelveticaNeue-MediumItalic", "HelveticaNeue-Bold", "HelveticaNeue-BoldItalic", "HelveticaNeue-CondensedBold", "HelveticaNeue-CondensedBlack"]
Family: Hiragino Maru Gothic ProN Font names: ["HiraMaruProN-W4"]
Family: Hiragino Mincho ProN Font names: ["HiraMinProN-W3", "HiraMinProN-W6"]
Family: Hiragino Sans Font names: ["HiraginoSans-W3", "HiraginoSans-W5", "HiraginoSans-W6", "HiraginoSans-W7", "HiraginoSans-W8"]
Family: Hoefler Text Font names: ["HoeflerText-Regular", "HoeflerText-Italic", "HoeflerText-Black", "HoeflerText-BlackItalic"]
Family: Impact Font names: ["Impact"]
Family: Kailasa Font names: ["Kailasa", "Kailasa-Bold"]
Family: Kefa Font names: ["Kefa-Regular"]
Family: Khmer Sangam MN Font names: ["KhmerSangamMN"]
Family: Kohinoor Bangla Font names: ["KohinoorBangla-Regular", "KohinoorBangla-Light", "KohinoorBangla-Semibold"]
Family: Kohinoor Devanagari Font names: ["KohinoorDevanagari-Regular", "KohinoorDevanagari-Light", "KohinoorDevanagari-Semibold"]
Family: Kohinoor Gujarati Font names: ["KohinoorGujarati-Regular", "KohinoorGujarati-Light", "KohinoorGujarati-Bold"]
Family: Kohinoor Telugu Font names: ["KohinoorTelugu-Regular", "KohinoorTelugu-Light", "KohinoorTelugu-Medium"]
Family: Lao Sangam MN Font names: ["LaoSangamMN"]
Family: Malayalam Sangam MN Font names: ["MalayalamSangamMN", "MalayalamSangamMN-Bold"]
Family: Marker Felt Font names: ["MarkerFelt-Thin", "MarkerFelt-Wide"]
Family: Menlo Font names: ["Menlo-Regular", "Menlo-Italic", "Menlo-Bold", "Menlo-BoldItalic"]
Family: Mishafi Font names: ["DiwanMishafi"]
Family: Mukta Mahee Font names: ["MuktaMahee-Regular", "MuktaMahee-Light", "MuktaMahee-Bold"]
Family: Myanmar Sangam MN Font names: ["MyanmarSangamMN", "MyanmarSangamMN-Bold"]
Family: Noteworthy Font names: ["Noteworthy-Light", "Noteworthy-Bold"]
Family: Noto Nastaliq Urdu Font names: ["NotoNastaliqUrdu", "NotoNastaliqUrdu-Bold"]
Family: Noto Sans Kannada Font names: ["NotoSansKannada-Regular", "NotoSansKannada-Light", "NotoSansKannada-Bold"]
Family: Noto Sans Myanmar Font names: ["NotoSansMyanmar-Regular", "NotoSansMyanmar-Light", "NotoSansMyanmar-Bold"]
Family: Noto Sans Oriya Font names: ["NotoSansOriya", "NotoSansOriya-Bold"]
Family: Optima Font names: ["Optima-Regular", "Optima-Italic", "Optima-Bold", "Optima-BoldItalic", "Optima-ExtraBlack"]
Family: Palatino Font names: ["Palatino-Roman", "Palatino-Italic", "Palatino-Bold", "Palatino-BoldItalic"]
Family: Papyrus Font names: ["Papyrus", "Papyrus-Condensed"]
Family: Party LET Font names: ["PartyLetPlain"]
Family: PingFang HK Font names: ["PingFangHK-Regular", "PingFangHK-Ultralight", "PingFangHK-Thin", "PingFangHK-Light", "PingFangHK-Medium", "PingFangHK-Semibold"]
Family: PingFang SC Font names: ["PingFangSC-Regular", "PingFangSC-Ultralight", "PingFangSC-Thin", "PingFangSC-Light", "PingFangSC-Medium", "PingFangSC-Semibold"]
Family: PingFang TC Font names: ["PingFangTC-Regular", "PingFangTC-Ultralight", "PingFangTC-Thin", "PingFangTC-Light", "PingFangTC-Medium", "PingFangTC-Semibold"]
Family: Rockwell Font names: ["Rockwell-Regular", "Rockwell-Italic", "Rockwell-Bold", "Rockwell-BoldItalic"]
Family: STIX Two Math Font names: ["STIXTwoMath-Regular"]
Family: STIX Two Text Font names: ["STIXTwoText", "STIXTwoText-Italic", "STIXTwoText_Medium", "STIXTwoText-Italic_Medium-Italic", "STIXTwoText_SemiBold", "STIXTwoText-Italic_SemiBold-Italic", "STIXTwoText_Bold", "STIXTwoText-Italic_Bold-Italic"]
Family: Savoye LET Font names: ["SavoyeLetPlain"]
Family: Sinhala Sangam MN Font names: ["SinhalaSangamMN", "SinhalaSangamMN-Bold"]
Family: Snell Roundhand Font names: ["SnellRoundhand", "SnellRoundhand-Bold", "SnellRoundhand-Black"]
Family: Symbol Font names: ["Symbol"]
Family: Tamil Sangam MN Font names: ["TamilSangamMN", "TamilSangamMN-Bold"]
Family: Thonburi Font names: ["Thonburi", "Thonburi-Light", "Thonburi-Bold"]
Family: Times New Roman Font names: ["TimesNewRomanPSMT", "TimesNewRomanPS-ItalicMT", "TimesNewRomanPS-BoldMT", "TimesNewRomanPS-BoldItalicMT"]
Family: Trebuchet MS Font names: ["TrebuchetMS", "TrebuchetMS-Italic", "TrebuchetMS-Bold", "Trebuchet-BoldItalic"]
Family: Verdana Font names: ["Verdana", "Verdana-Italic", "Verdana-Bold", "Verdana-BoldItalic"]
Family: Zapf Dingbats Font names: ["ZapfDingbatsITC"]
Family: Zapfino Font names: ["Zapfino"]
```

If you have a font in your Font Book that doesn't appear on the list, it won't just work with SpriteKit. You have to explicitly add it to Xcode. Apple [has a page](https://developer.apple.com/documentation/uikit/text_display_and_fonts/adding_a_custom_font_to_your_app#) on how to do that. To clarify further, here are screenshots made with Xcode 15.4. Before making these modifications to the info tab of your project target, make sure to add the font to Xcode. You have to add the font **as a file**, **not as an image** inside Assets.

<img src="Screenshots/Xcode add font - 1 Target info.png" alt="Xcode add font - 1 Target info" style="width:100%;" />

<img src="Screenshots/Xcode add font - 2 Fonts provided by application.png" alt="Xcode add font - 2 Fonts provided by application" />

<img src="Screenshots/Xcode add font - 3 Fonts provided by application.png" alt="Xcode add font - 3 Fonts provided by application" />

<img src="Screenshots/Xcode add font - 4 Add font name.png" alt="Xcode add font - 4 Add font name" />

## Centering a node inside another one

*12 March 2024*

For any nodes `nodeToCenter` and `referenceNode`, you can center one relative to another with:

```swift
nodeToCenter.position = CGPoint(x: referenceNode.frame.midX, y: referenceNode.frame.midY)
```

There are special cases where a node will be automatically centered. If your node to center has a parent, and if the parent has a property of `anchorNode`, such as `SKScene` and `SKSpriteNode`, then any child node of those with position ``CGPoint(x: 0, y: 0)` are automatically centered relative to their parent node.

## Visualize the bounding box of a node

*9 March 2024*

You can use the `calculateAccumulatedFrame()` method on the node you want to visualize, and draw a `SKShapeNode` around it:

```swift
func visualizeFrame(for targetNode: SKNode, in scene: SKScene) {
    // Unique name for the visualization node to easily identify it
    let visualizationNodeName = "visualizationFrameNode"

    // Check if the visualization node already exists
    let existingVisualizationNode = scene.childNode(withName: visualizationNodeName) as? SKShapeNode

    var frame: CGRect = targetNode.calculateAccumulatedFrame()
    let path = CGPath(rect: frame, transform: nil)

    if let visualizationNode = existingVisualizationNode {
        // Update the existing node's path to reflect the new frame
        visualizationNode.path = path
    } else {
        // Create a new SKShapeNode for the frame if it doesn't exist
        let frameNode = SKShapeNode(path: path)
        frameNode.name = visualizationNodeName
        frameNode.strokeColor = SKColor.red // Color of the frame
        frameNode.lineWidth = 2 // Width of the frame lines
        frameNode.zPosition = 100 // Ensure it's visible above other nodes

        // Add the new visualization node to the scene
        scene.addChild(frameNode)
    }
}
```

You can either call that function one time, or you can call it every frame. Interesting to note: the way you call your function every time impacts how the bounding box will be draw. If your target node is affected by physics, and you call `visualizeFrame` in `update`, then the bounding box will lag behind your moving physical object, because `update` is called before physics are simulated.

A better way to accurately track your moving physical object is to use the `didSimulatePhysics` function of SpriteKit's game loop:

```swift
override func update(_ currentTime: TimeInterval) {
    // this draws a shape that is lagging behind the physical object
    visualizeFrame(for: effectNode, in: self)
}

override func didSimulatePhysics() {
    // didSimulatePhysics is the last function called before rendering
    // tracking physical behavior is most accurate here
    visualizeFrame(for: effectNode, in: self)
}
```

## zPosition

*7 March 2024*

> The drawing order influenced by zPosition is also relative to the node hierarchy. A node with a higher zPosition will appear in front of nodes with lower zPosition values, but this is within the context of their respective parent nodes. If a parent node has a lower zPosition than another node, all of its children will also appear below that node, regardless of their individual zPositions.

ChatGPT 4, *accessed 7  March 2024*

> SpriteKit uses the zPosition value only to determine the hit testing and drawing order. You can also the z position to implement your own game effects. For example, you might use the height of a node to determine how it is rendered or how it moves onscreen. In this way, you can simulate fog or parallax effects. SpriteKit does not create these effects for you. Usually, you implement them by processing the scene immediately before it is rendered.

[Apple Documentation](https://developer.apple.com/documentation/spritekit/sknode/1483107-zposition), *accessed 7  March 2024*

## Get the texture of a node

*6 March 2024*

In SpriteKit, you can get a texture from any node, including a collection of nodes, which you can group under the same `SKNode`.

For example, you could define a node using `SKShapeNode`, get the texture of that node, and create a `SKSpriteNode` with the texture.

Similarly, you can get the resulting texture of many nodes at one, 

```swift
if let nodeTexture = view.texture(from: myNode) {
    myNode.physicsBody = SKPhysicsBody(texture: nodeTexture, alphaThreshold: 0.6, size: nodeTexture.size())
    myNode.physicsBody?.density = 100
    myNode.physicsBody?.friction = 1
}
```

## Marching ants

*6 March 2024*

Marching ants is the effect that you may see when you draw a selection rectangle, where the borders of the rectangle are made of dashes running along the border. With SpriteKit, you can draw a shape with dashed borders like this:

```swift
let boundingBox = SKShapeNode(rectOf: CGSize(width: 200, height: 100))
boundingBox.lineWidth = 2
boundingBox.strokeColor = .blue
boundingBox.fillColor = .clear
// the magic line
boundingBox.path = containerBoundingBox.path?.copy(dashingWithPhase: 0, lengths: [10,10])
addChild(boundingBox)
```

Play along the values inside the array that is passed to `lengths` to get various dashes lengths and sequences. You can have multiple dashes and gaps in the array.

In order to animate the dashes and achieve the marching ants effect, increment the `dashingWithPhase` parameter across time:

```swift
let myPath = CGPath(rect: CGRect(x: 0, y: 0, width: 200, height: 200), transform: nil)
var phase: CGFloat = 0
let dashPattern: [CGFloat] = [10, 5]
let dashedPath = myPath.copy(dashingWithPhase: phase, lengths: dashPattern)
let boundingBox = SKShapeNode(path: dashedPath)
boundingBox.lineWidth = 3
boundingBox.strokeColor = .systemBlue
boundingBox.fillColor = .clear
boundingBox.position = .zero

let incrementDashingPhaseAction = SKAction.run {
    phase += 1
    let newDashedPath = myPath.copy(dashingWithPhase: phase, lengths: dashPattern)
    boundingBox.path = newDashedPath
}

let waitAction = SKAction.wait(forDuration: 0.02)
let sequenceAction = SKAction.sequence([incrementDashingPhaseAction, waitAction])
let repeatForeverAction = SKAction.repeatForever(sequenceAction)
boundingBox.run(repeatForeverAction)
addChild(boundingBox)
```

## 9 parts slicing

*5 March 2024*

With this, you can scale sprites without distorting them. The idea is that you slice the sprite's texture into 9 parts: a grid of 3x3. Only the central part is stretched vertically and horizontally when the sprite is scaled.

```swift
let button = SKSpriteNode(imageNamed: "my_texture@2x")
button.centerRect = CGRect(x: 12.0/28.0,
                           y: 12.0/28.0,
                           width: 4.0/28.0,
                           height: 4.0/28.0)
```

Once you pass in a value to the `centerRect` property of a sprite node, that sprite node will be sliced. The `CGRect` you pass in is the central rectangle the 3x3 slicing grid. It is positioned relative to the sprite node. Its positioning (x,y) and sizing (width, height) takes values from 0 to 1. Therefore, by convention, you fill the values by dividing your desired target by one size of your texture.

For example, if your texture is 100 points wide and 60 points high, your x and width values will be `desiredValue/100`.

Here's a helper function that gets you that center rectangle given a width and height of the corner, and the sprite node to slice:

```swift
/// Calculates the CGRect for the center part of a 9-slice sprite.
/// - Parameter cornerWidth: The width of the corner parts
/// - Parameter cornerHeight: The height of the corner parts
/// - Parameter sprite: The SKSpriteNode for which to calculate the center rect
/// - Returns: A CGRect representing the center rectangle for 9-slice scaling
func setCenterRect(cornerWidth: CGFloat, cornerHeight: CGFloat, sprite: SKSpriteNode) -> CGRect {
    guard let textureSize = sprite.texture?.size() else {
        return .zero
    }
    
    let totalWidth = textureSize.width
    let totalHeight = textureSize.height
    
    let centerSliceWidth = totalWidth - (cornerWidth * 2)
    let centerSliceHeight = totalHeight - (cornerHeight * 2)
    
    let centerSliceRect = CGRect(x: cornerWidth / totalWidth,
                                 y: cornerHeight / totalHeight,
                                 width: centerSliceWidth / totalWidth,
                                 height: centerSliceHeight / totalHeight)
    
    return centerSliceRect
}
```

See [Resizing a Sprite in Nine Parts](https://developer.apple.com/documentation/spritekit/skspritenode/resizing_a_sprite_in_nine_parts) on Apple Documentation.

## Touch events

*5 March 2024*

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    // safety precaution to make sure there was a touch event captured
    guard let touch = touches.first else { return }
    
    // get touch coordinates relative to the window
    let location = touch.location(in: nil)
    
    // get touch coordinates relative to SpriteKit's scene
    // make sure the scene is not an optional.
    // You can get touch coordinates relative to any container passed to `in`
    let location = touch.location(in: scene)
}
```

## Multi touch

*5 March 2024*

In SpriteKit, multi touch is not enabled by default. Add this to your view setup inside `SKScene`, typically in the `didMove` method:

```swift
override func didMove(to view: SKView) {
    view.isMultipleTouchEnabled = true
}
```

Trivia: on iPhone, it seems that the maximum supported amount of simultaneous touch points is 5.

## Setup with SwiftUI

```swift
import SwiftUI
import SpriteKit

// SwiftUI
struct ContentView: View {
    var myScene = MyScene()
    var body: some View {
        SpriteView(scene: myScene)
    }
}

// SpriteKit
class MyScene: SKScene {
    
}
```

If you need to bind data and methods between SwiftUI and SpriteKit, you can use the Observation framework (iOS 17+). Observation is set up by importing `Observation`, wrapping the instance of the SpriteKit scene with `@State`, and adding the `@Observable` macro before my SpriteKit class declaration:

```swift
import SwiftUI
import SpriteKit
import Observation

// SwiftUI
struct SpriteKitSwiftUI: View {
    @State var myScene = MyScene()
    var body: some View {
        SpriteView(scene: myScene)
        Button(action: {
            myScene.doSomething()
        }, label: {
            Text("Toggle")
        })
    }
}

// SpriteKit
@Observable class MyScene: SKScene {
    
    func doSomething() {
        
    }
    
}
```

## SpriteView configuration

SpriteKit was made in the era of UIKit. A SpriteKit view, `SKView`, was configured inside a `UIViewController`. With SwiftUI, configuring a SpriteKit view is now done with a configuration object that you pass to SpriteView:

```swift
SpriteView(
    // an instance of a SKScene
    scene: myScene,
    // between 1 and 120
    preferredFramesPerSecond: 60,
    // some display options
    options: [.ignoresSiblingOrder, .allowsTransparency, .shouldCullNonVisibleNodes],
    // some debug helpers
    debugOptions: [.showsFPS, .showsNodeCount, .showsDrawCount, .showsQuadCount, .showsPhysics, .showsFields]
)
```

## Scene size

Most tutorials and resources available on SpriteKit will use a specific size for the scene, and pass it to the initializer method. What size should you use in your own SpriteKit project?

Scene size in SpriteKit is a convenience property. In reality, SpriteKit scenes are infinitely large. Users specify a scene size because 1) they have fixed size art that has to fit in specific places within a fixed screen ratio, and 2) they use the scene size to place objects relative to these device-derived bounds.

*21 January 2024*

How should I size my SpriteKit scene? How does scene size affect performance? Will my nodes' position be constrained by the scene size? If a physics body falls under gravity, will it continue to fall indefinitely?

If we command-click on `SKScene` in a SpriteKit code, we can bring up the header information for the class:

> A scene is infinitely large, but it has a viewport that is the frame through which you present the content of the scene. The passed in size defines the size of this viewport that you use to present the scene.

An older version of the header, [quoted here](https://stackoverflow.com/a/33447352/420176), reads:

> To display different portions of your scene, move the contents relative to the viewport. One way to do that is to create a SKNode to function as a viewport transformation. That node should have all visible contents parented under it.

That version probably predates the introduction of `SKCameraNode`, since a SpriteKit camera does essentially that. Now regarding the relation between the scene size and the viewport size, we can read this in the same header file:

> fill: Scale the SKScene to fill the entire SKView
>
> aspectFill: Scale the SKScene to fill the SKView while preserving the scene's aspect ratio. Some cropping may occur if the view has a different aspect ratio.
>
> aspectFit: Scale the SKScene to fit within the SKView while preserving the scene's aspect ratio. Some letterboxing may occur if the view has a different aspect ratio.
>
> resizeFill: Modify the SKScene's actual size to exactly match the SKView.

So a SpriteKit scene is an infinite canvas by default. The part of the scene that is being drawn and rendered is the view (`SKView`). A scene can either be scaled to fit a view (one of the 4 `scaleMode`), or be drawn through a camera frame that determines which crop of the scene is in view.

Regardless of scene size, objects can be positioned freely without limit. Does positioning objects tens or hundreds of thousands of points away from each other have an impact on memory consumption and performance? I don't know yet.

## Scene configuration

Regardless of the UI framework you are using, additional scene and view setup can be made within the SpriteKit class.

```swift
class MyScene: SKScene {
    
    // Use sceneDidLoad for one-time setup
    override func sceneDidLoad() {
        self.scaleMode = .resizeFill
        // anchor point is a convenience property
        self.anchorPoint = CGPoint(x: 0.5, y: 0.5)
        self.physicsWorld.gravity = CGVector(dx: 0, dy: -9.8)
        self.physicsBody?.usesPreciseCollisionDetection = false
        self.physicsWorld.speed = 1
    }

    // Use didMove for code that should run whenever the scene is actually in view
    // A scene could be in memory even if it's not presented, and thus, didMove will execute its code again when the scene will get displayed.
    override func didMove(to view: SKView) {
        // some encapsulated setup method
        setupScene()
        // play/pause state
        self.isPaused = isScenePaused
        // background color of the scene
        self.backgroundColor = .clear
        // enable multi-touch event handling
        view.isMultipleTouchEnabled = true
    }
    
}
```

## Setup with UIKit

```swift
import UIKit
import SpriteKit

class GameViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let scene = GameScene(size: CGSize(width: 2048, height: 1536))
        scene.scaleMode = .aspectFill

        let view = self.view as! SKView
        view.presentScene(scene)
    }
}

class GameScene: SKScene {
    override init(size: CGSize) {
        super.init(size: size)
    }

    required init(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func didMove(to view: SKView) {

    }
}

// load a scene from file
import UIKit
import SpriteKit

class GameViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        if let view = self.view as! SKView? {
            if let scene = SKScene(fileNamed: "SpriteKitScene") {
                scene.scaleMode = .aspectFill
                view.presentScene(scene)
            }
        }
    }
}
```

## UIKit view configuration

```swift
// inside the view controller
view.preferredFramesPerSecond = 60
view.showsFPS = true
view.ignoresSiblingOrder = false
view.showsNodeCount = true
view.showsPhysics = true
scene.scaleMode = .aspectFill
```

## SKScene methods

``` swift
class scene: SKScene{
    // declare a variable, with a type, without value
    var myNodePosition: CGPoint?

    override init(size: CGSize) {
        super.init(size: size)
        // the scene size is the passed size
    }

    override func didMove(to view: SKView) {
        scene?.scaleMode = .aspectFit
        // the scene size will depend on scaling mode
    }

    override func update(_ currentTime: TimeInterval) {
        print(currentTime) // time since app started
    }
}

// scene transition
class StartView: SKScene {
    let sceneToTransitionTo = MainView(size: size)
    sceneToTransitionTo.scaleMode = .aspectFill
    let transitionEffect = SKTransition.push(with: .down, duration: 0.5)
    view?.presentScene(sceneToTransitionTo, transition: transitionEffect)  
    }

    class MainView: SKScene {
    // 
}
```

## Node creation

```swift
// empty node
let parentNode = SKNode()

// sprite from image
var mySprite = SKSpriteNode(fileNamed: "imageFile") // add the image to Xcode assets
mySprite.anchorPoint = CGPoint(x: 0.5, y: 0.5)
mySprite.position = CGPoint.zero

// text
let myText = SKLabelNode()
myText.text = "Text to display"
myText.fontSize = 28
myText.fontName = "Impact"
myText.fontColor = SKColor.black
myText.position = CGPoint(x: size.width/2, y: size.height/2)

// shape
let cornerRadius: CGFloat = 7
let rectSize = CGSize(width: 60, height: 60)
let roundedRectPath = UIBezierPath(roundedRect: CGRect(origin: CGPoint(x: 0, y: 0), size: rectSize), cornerRadius: cornerRadius)
shapeObject = SKShapeNode(path: roundedRectPath.cgPath)
shapeObject.fillColor = .red

// hollow shape with border
let myFrame = SKShapeNode(rectOf: CGSize(width: 60, height: 60))
myFrame.fillColor = .clear
myFrame.strokeColor = .red
```

## Node properties

```swift
// position
myNode.position = CGPoint(x: 350, y: 400)
myNode.position.x = CGFloat
myNode.position.y = CGFloat
myNode.zPosition = Int // z-index

// rotate
myNode.zRotation = .pi / 4 // CGFloat, in this case, π/4, which is 45 deg counterclockwise

// scale
myNode.scale = 1 // Read only
myNode.scaleX = 1 // x axis only
myNode.scaleY = 1 // y axis only
myNode.setScale(1) // setScale is a convenience method 
myNode.setScale(x: 1.5, y: 0.5) // setScale is a convenience method

// Rendering
myNode.alpha = CGFloat

// methods
myNode.isHidden = false // Boolean. When hidden, a node and its descendants are not rendered. However, they still exist in the scene and continue to interact in other ways. For example, the node’s actions still run and the node can still be intersected with other nodes.

frame
frame.midX
frame.midY
```

## Physics

```swift
// scene properties. Declare in init()
myScene.physicsWorld.gravity = CGVector(dx: 0, dy: -9.8) // set gravity
myScene.physicsBody = SKPhysicsBody(edgeLoopFrom: scene.frame) // bodies collide with the edges of the scene

myNode.physicsBody = SKPhysicsBody()
myNode.physicsBody!.restitution = 0.2 // default, bounciness, [0, 1]
myNode.physicsBody!.friction = 0.2 // default, roughness, [0, 1]
myNode.physicsBody!.density = 1.0 // default, kg/m2
myNode.physicsBody!.isDynamic = Bool // can be moved by physics or not
myNode.physicsBody!.allowsRotation = Bool
myNode.physicsBody!.mass = 1.0 // automatically derived from density and area
myNode.physicsBody!.area // read only
myNode.physicsBody!.node // real only, returns the parent node
myNode.physicsBody!.linearDamping = 0.1 // default, [0, 1]
myNode.physicsBody!.angularDamping = 0.1 // default, [0, 1]
myNode.physicsBody!.affectedByGravity = Bool // only with regard to scene gravity
myNode.physicsBody!.pinned = Bool // Stick position to parent. Rotation still allowed
myNode.physicsBody!.velocity = CGVector // m/s
myNode.physicsBody!.angularVelocity = CGVector // radians/s
myNode.physicsBody!.isResting = Bool

// compound bodies
let bodies = [node1, node2]
compound.physicsBody = SKPhysicsBody(bodies: bodies) // Creates a physics body that's shaped like a union of the supplied array of bodies.

// physics forces
myNode.physicsBody?.applyImpulse(CGVector(dx: 10, dy: 0)) // N/s
myNode.physicsBody?.applyForce(CGVector(dx: 10, dy: 0)) // N/s, acceleration is applied every simulation step

// constraints
let rotationConstraint = SKConstraint.zRotation(SKRange(lowerLimit: -.pi/4, upperLimit: .pi/4))
myNode.constraints = [rotationConstraint]

let range = SKRange(lowerLimit: 0.0, upperLimit: 0.0)
let orientConstraint = SKConstraint.orient(to: targetNode, offset: range)
myNode.constraints = [orientConstraint]

// spring joint
let spring = SKPhysicsJointSpring.joint(
    withBodyA: physicsBody!, // connect with the edges of the scene itself
    bodyB: nodeA.physicsBody!,
    anchorA: position, // the scene position
    anchorB: nodeB.position
)
spring.frequency = 0.5
spring.damping = 0.2
scene.physicsWorld.add(ropeJoint)

// pin joint
let myNode = scene.childNode(withName: "myNode") as? SKSpriteNode
let pinPoint = self.convert(myNode!.anchorPoint, to: scene)

let myNodePin = SKPhysicsJointPin.joint(withBodyA: scene.physicsBody!, bodyB: myNode!.physicsBody!, anchor: pinPoint)
scene.physicsWorld.add(myNodePin)

// methods of a SpriteKit scene class
override func didSimulatePhysics() {
    // run code after physics simulation
}
```

## Actions

```swift
// Scale
let appear = SKAction.scale(to: 1.0, duration: 0.5) // scale(to) can not be reversed
myNode.run(appear)

let scaleUp = SKAction.scale(by: 1.2, duration: 0.25) // scale(by) can be reversed
let scaleDown = scaleUp.reversed()

// Position
let moveToAction = SKAction.move(to: CGPoint(x: 1.0, y: 1.0), duration: 1.0)

let moveByAction = SKAction.moveBy(CGPoint(x: 1.0, y: 1.0), duration: 1.0)
let reverseMoveBy = moveByAction.reversed() // moveBy actions can be reversed

//
SKAction.wait(forDuration: 1.0) // duration in sec

//
SKAction.resize(byWidth: -width, height: -height, duration: sec) // reversible

//
let myCustomAction = SKAction.customAction(withDuration: duration) { node, elapsedTime in
    // arbitrary code that runs over a duration
    // `node`, `duration`, and `elapsedTime` are values
}

// sequences: running actions one after another
let mySequence = SKAction.sequence([myAction1, myAction2, myAction3])
myNode.run(mySequence)

let arbitraryCode = SKAction.run{ /*code to execute*/ }

// groups: running actions at the same time
let scaleNode = SKAction.scale(by: 1.2, duration: 0.25)
let rotateNode = SKAction.rotate(byAngle: -.pi/2, duration: 1.0)
let groupActions = SKAction.group([scaleNode, rotateNode])
myNode.run(groupActions)

// repeat
let repeatActionForever = SKAction.repeatForever(myAction)
let repeatActionTwoTimes = SKAction.repeat(myAction, count: 2)
myNode.run(myAction)

// sprite animation
let spriteAnimation: SKAction
var sprites:[SKTexture] = []
for i in 1...6 {
    sprites.append(SKTexture(imageNamed: "arrow\(i)")) // as many i as sprites
}
spriteAnimation = SKAction.animate(with: sprites, timePerFrame: 0.25)
myNode.run(SKAction.repeatForever(spriteAnimation))
```

## Sound

```swift
// sound
// put the sound file in a "Sounds" folder in the project
run(SKAction.playSoundFileNamed("hitCircle.wav", waitForCompletion: false))
```

## Camera

```swift
// camera
let cameraNode = SKCameraNode()
addChild(cameraNode)
camera = cameraNode
cameraNode.position = CGPoint(x: size.width/2, y: size.height/2)
```

## Events

```swift
// Whether a node receives touch events. Default = false
myNode.isUserInteractionEnabled = Bool

// touch began
override func touchesBegan(_ touches: Set<UITouch>,  with event: UIEvent?) {
    for touch in touches {
        let touchIdentifier = touch.hashValue

        // Handle the touch based on its identifier
        switch touchIdentifier {
            case touches.first?.hashValue:
            print("First touch")
            case touches.prefix(3).last?.hashValue:
            print("Third touch")
            default:
            print("Another touch")
        }
    }
}

// hit detection
let touchLocation = touch.location(in: self)
if myNode.frame.contains(touchLocation) {
    // myNode was touched
}
```

---

## Examples

```swift
// Continuously rotate a node
func continuouslyRotate(_ node: SKNode) {
    let rotateAction = SKAction.rotate(byAngle: .pi * 2, duration: 10.0)
    let continuousRotation = SKAction.repeatForever(rotateAction)
    node.run(continuousRotation)
}
```

```swift
// Add a frame around a sprite
let aSprite = SKSpriteNode(imageNamed: "cloud")
let theFrame = SKShapeNode(rectOf: aSprite.size)
theFrame.strokeColor = .systemBlue
theFrame.lineWidth = 3
aSprite.addChild(theFrame)
addChild(aSprite)
```

```swift
// Animating a CIFilter over a duration
let animationDuration = 3.0
let myRotationAction = SKAction.customAction(withDuration: animationDuration) { node, elapsedTime in
        let dynamicValue = 40 * elapsedTime
        effectNode.filter = ChainCIFilter(filters: [
            CIFilter(name: "CIMotionBlur", parameters: ["inputRadius": dynamicValue]),
        ])
}
let repeatAction = SKAction.repeatForever(myRotationAction)
effectNode.run(repeatAction)
```

```swift
// Add a physical object to the scene
func addPhysicalObject() {
    let physicalBox = SKSpriteNode(color: .red, size: CGSize(width: 50, height: 50))
    physicalBox.position = CGPoint.zero
    physicalBox.physicsBody = SKPhysicsBody()
    physicalBox.physicsBody!.applyForce(CGVector(dx: 0, dy: 100))
    physicalBox.physicsBody?.applyImpulse(CGVector(dx: 100, dy: 1000))
    addChild(physicalBox)
}
```

```swift
// Scene with transparent background
import SwiftUI
import SpriteKit

struct transparentSpriteView: View {
    var myScene = SpriteKitScene(size: CGSize(width: 1000, height: 1000))
    
    var body: some View {
        ZStack {
            SpriteView(
                scene: myScene,
                options: [.allowsTransparency] // Important bit Nº1
            )
                .ignoresSafeArea()
                .background(.blue)
        }
    }
}

class SpriteKitScene: SKScene {  
    override init(size: CGSize) {
        super.init(size: size)
        self.scaleMode = .resizeFill
        self.anchorPoint = CGPoint(x: 0.5, y: 0.5)
    }
    
    required init(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func didMove(to view: SKView) {
        backgroundColor = .clear // Important bit Nº2
    }
}
```

## Links & Resources

- 🎬 [How to use SKLightNodes in Swift](https://www.youtube.com/watch?v=SW2G0bQ2iUo), *accessed 13 March 2024*
- 📝 [Outline text in SpriteKit](http://endpoint.nl/blog/outline-text-in-spritekit)
- 📝 [The history of Cocos2d in a glimpse](https://retro.moe/2017/04/16/cocos2d-in-a-glimpse/), 2017. *accessed 8 March 2024*
- 📝 [SpriteKit and UITextView](https://forums.developer.apple.com/forums/thread/7424)
- 🎬 [Haskell SpriteKit — A Purely Functional API for a Stateful Animation System & Physics Engine](https://speakerdeck.com/mchakravarty/haskell-spritekit-a-purely-functional-api-for-a-stateful-animation-system-and-physics-engine)
- 📝 [Some things I've learned working with SpriteKit/GameplayKit](https://www.reddit.com/r/spritekit/comments/hw30kv/some_things_ive_learned_working_with/), Reddit
- 📝 [Fast way to blur the background behind a Sprite Node](https://forums.developer.apple.com/forums/thread/112334), Apple Forums
- 📝 [SpriteKit posts by KnightOfDragon](https://stackoverflow.com/search?q=user:2709645+[sprite-kit]), StackOverflow
- 📝 [Resizing a Sprite in Nine Parts](https://developer.apple.com/documentation/spritekit/skspritenode/resizing_a_sprite_in_nine_parts), slice a node's texture in 3x3 programmatically.
- 📝 [SpriteKit Physics](https://medium.com/@jjacobson/spritekit-physics-14331398b308), 2017.
- 📝 [A Guide to SpriteKit Actions](https://medium.com/hackernoon/a-guide-to-spritekit-actions-c20b079f5398), 2017
- 📝 Written tutorial, [controlling a SpriteKit element with a SwiftUI controller](https://munirwanis.github.io/blog/2020/wwdc20-spritekit-swiftui), 2020.
- 📝 Written tutorial, [Creating the Classic "Snake" Game with SpriteKit](https://hackernoon.com/creating-the-classic-snake-game-with-spritekit), 2023.
- 🎬 Video course, [SpriteKit, Protocols, App and ViewController LifeCycles](https://podtail.com/podcast/ios-application-development-17/s05-spritekit-protocols-app-and-viewcontroller-lif/), 2017.
- SpriteKit + SwiftUI: https://www.youtube.com/watch?v=yR15IqAjvC4
- https://github.com/kodecocodes/SKTUtils
- SwiftUI + SpriteKit: https://youtube.com/watch?v=nduPV7-3NtM&t=399
- 🔈 [Building 3D Apps with SceneKit](https://www.mergeconflict.fm/184), Merge Conflict podcast

---

This file started on 7 July 2023 as a collection of ready-to-use boilerplates called "SpriteKit Cheatsheet". In March 2024, it became a list of timestamped notes about topics related to SpriteKit.
