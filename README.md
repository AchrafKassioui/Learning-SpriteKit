# Learning SpriteKit

## To do

- Could we use `SKTexture(rect:in:)` to create a multi-body physics compound from the texture of a label node? Idea while watching [Apple's SpriteKit introduction video](https://devstreaming-cdn.apple.com/videos/wwdc/2013/502xex3x2iwfiaeglpjw0mh54u/502/502-HD.mov), 15:15, *12 April 2024*
- Write about `anchorPoint` for `SKSpriteNode` and look up `usesMipmaps`. *15 March 2024*

## SpriteKit is not siloed

*16 April 2024*

I thoroughly enjoyed watching SpriteKit introduction video in WWDC 2013, session 502. A link to an [HD version of the video can be found here](http://devstreaming-cdn.apple.com/videos/wwdc/2013/502xex3x2iwfiaeglpjw0mh54u/502/502-HD.mov?dl=1). Tim Oriol, SpriteKit lead engineer, mentions something in passing that was quite thought provoking to me: that we could use our own physics engine with SpriteKit, instead of SpriteKit built-in engine.

My takeaway is that SpriteKit is rather a toolkit framework meant to work with other frameworks, especially other Apple frameworks. SpriteKit also isn't very opinionated about how to structure and organize your code. It's up to you to design your own logic that uses SpriteKit as one of its components. It seems to me that SpriteKit was not designed to be a siloed environment, the way other game engines usually are.

Another interesting example is a macOS app I tested recently, called [Euler VS Pro](https://www.eulervs.com). Euler Visual Synthesizer uses a SpriteKit view to visualize shapes...in 3D! SpriteKit is used as the renderer, i.e. the end point rasterizer, while the 3D projections are taken care of inside the app's own logic, [using simd vector operations](https://discord.com/channels/1119028615733067808/1119028616844562463/1228520361285648465) on the CPU.

## SpriteKit original team

*9 April 2024*

Below is a list of people from Apple who were involved with the development of SpriteKit.

- **[Tim Oriol](https://www.linkedin.com/in/toriol/)**: "Original architect and engineering lead for SpriteKit, shipped in iOS 7. Responsible for initial proposal, prototyping and research, production architecture and feature implementation, growing the team, and designing the final API surface. [...] Continued to lead and grow the SpriteKit team. Responsible for migrating the rendering system to Metal and new features including animatable mesh warps, camera system, and custom shader support. Designed and implemented the particle effects system to support the fireworks fullscreen message effects (shipped in iOS 10)." [ndr: this is one of the reasons I'm interested in SpriteKit and native frameworks: I want seamless integration with the operating system and the rest of the features that users of that platform are accustomed to.]
- [Jacques Gasselin de Richebourg](https://www.linkedin.com/in/jacques-gasselin-de-richebourg-3b8924108/)

## View properties

*8 April 2024*

Within your SpriteKit code, you can access some interesting properties of the view containing the SpriteKit scene.

```swift
override func didMove(to view: SKView) {
    /// modify the anchor point of the SKView itself
    view.anchorPoint = CGPoint(x: 0.5, y: 0.5)
    
    /// access the Core Animation properties of the layer containing the view
    view.layer.borderWidth = 5
    view.layer.cornerRadius = 44
    view.layer.cornerCurve = .continuous
    view.layer.borderColor = SKColor.red.cgColor
```

## Texture filtering

*27 March 2024*

Texture filtering is a form of anti-aliasing. With sprite nodes, i.e. nodes that are explicitly made of bitmap data, SpriteKit's renderer provides two texture filtering modes: 

```swift
mySpriteNode.texture?.filteringMode = .linear // linera filtering, default mode
mySpriteNode.texture?.filteringMode = .nearest // pixelated mode
```

<img src="Screenshots/SpriteKit-filteringMode.png" alt="SpriteKit-filteringMode" style="width:50%;" />

For nodes that are drawn programmatically like `SKShapeNode`, SpriteKit provides an anti-aliasing switch that probably inherits from Core Graphics:

```swift
/// stroke edges and caps is smoothed (antialiased) when drawn
/// default mode = true
myShapeNode.isAntialiased = true
```

In the broader context of graphical authoring tools, it's interesting to pay attention at the anti-aliasing mode they use. Typically, when you zoom out in a program like Figma or Photoshop, the rendering is smoothed out, so you can still see points that are less than 1 pixel wide. If they weren't smoothed out, anything less than a full pixel would disappear when you zoom out beyond 100% zoom.

However, when you zoom in, these programs tends to disable smoothing, and they show you a representation of the pixels grid. This helps evaluate bitmap data as it is defined at the pixel level.

## Watch out

*26 March 2024*

These are peculiar behaviors I stumbled upon wile working with SpriteKit.

### Core Image filters can change the apparent behavior of a physical body

<video src="Screenshots/SpriteKit-BloomFilter-ON-showPhysics.mov" width="320"></video>

```swift
let colorWheeltexture = SKTexture(imageNamed: "color-wheel-sprite")
let colorWheel = SKSpriteNode(texture: colorWheeltexture)
colorWheel.physicsBody = SKPhysicsBody(texture: colorWheeltexture, size: colorWheeltexture.size())
colorWheel.physicsBody?.affectedByGravity = false

var bloomFilter = CIFilter.bloom()

let effectNode = SKEffectNode()
effectNode.filter = bloomFilter
bloomFilter.intensity = 1
bloomFilter.radius = 10
effectNode.addChild(colorWheel)

addChild(effectNode)

// ..

colorWheel.physicsBody?.angularVelocity += 5
```

Spinning a perfect circle that has a bloom filter applied to it can make the circle rock back and forth. It appears as if the sprite's center of rotation has changed. I experienced similar effects with other filters such as gaussian blur.

### Physics bodies can not be automatically generated from textures with holes in them

Examples: SKLabelNode with non contiguous characters, SKTexture with non contiguous opaque regions.

### Sprite node anchor point and physics body

Changing the anchor point of a sprite node does not reposition the physics body to follow the visual texture.

## Physics vs manual 

*25 March 2024*

SpriteKit physics engine changes the position and zRotation of nodes over time. The rate and direction at which it changes them is what is stored in the velocity and angular velocity properties. By combing that information with other physical informations, such as friction and rules of change, the physics engine calculates new positons and rotations and apply them to objects.

If you write code that changes the position or rotation of objects across time, for example through an SKAction or inside the update function, you are essentially writing your own physics engine, with your own rules.

*5 April 2024*

Numerically calculating the position of an object, by applying velocity each delta time, is called [explicit Euler](https://web.archive.org/web/20180823005957/https://gafferongames.com/post/integration_basics/).

## Camera and scene anchor point

*19 March 2024*

The scene anchor point is a convenience property that positions the origin of the scene relative to the view. By default, the scene's anchor point is (x: 0, y: 0). That puts the origin of the scene at the bottom left corner of the view. 

If you set the anchor point of the scene to (x: 0.5, y: 0.5), the origin of the scene will be located at the center point of the view. This way, when you create a node and add it to the scene, its position will be calculated relative to the center of the view, because the node is positioned relative to the origin of its parent, the scene, and the origin of the scene is at the center of the view. The scene's anchor point is a convenient way to start up your setup depending on your needs.

However, if you add a camera to the scene, the scene's origin will be positioned at the center of the camera view, regardless of the scene's anchor point—unless you reposition the camera, that is. So when the scene has a camera, the anchor point is ignored.

## Apple documentation errors

*19 March 2024*

The official documentation of SpriteKit is a must-read but it is unfortunately lacking. Some of it is actually false. Here are examples:

- `lineLength`: [Apple says](https://developer.apple.com/documentation/spritekit/skshapenode/1520398-linelength) that we can set values to that property. But in practice, this is a get only property. We can not use it to animate the drawing of a path.

## Core Graphics and SpriteKit

*18 March 2024*

One of the first things I wanted to do in SpriteKit is to customize the background. I wanted a background that looks like a grid. How could I do that? I could create an image with Pixelmator or Figma and import it in SpriteKit. But I needed to iterate quickly on the look of the grid, and generating large repetitive patterns is not that trivial with drawing software.

Suppose we want to generate a grid to display as a background in SpriteKit, how would we do it programmatically? We can use Core Graphics:

```swift
func generateGridTexture(cellSize: CGFloat, rows: Int, cols: Int) -> SKTexture? {
    /// Add 1 to the height and width to ensure the borders are within the sprite
    let size = CGSize(width: CGFloat(cols) * cellSize + 1, height: CGFloat(rows) * cellSize + 1)
    
    let renderer = UIGraphicsImageRenderer(size: size)
    let image = renderer.image { ctx in
        
        let bezierPath = UIBezierPath()
        let offset: CGFloat = 0.5
                                
        /// vertical lines
        for i in 0...cols {
            let x = CGFloat(i) * cellSize + offset
            bezierPath.move(to: CGPoint(x: x, y: 0))
            bezierPath.addLine(to: CGPoint(x: x, y: size.height))
        }
        /// horizontal lines
        for i in 0...rows {
            let y = CGFloat(i) * cellSize + offset
            bezierPath.move(to: CGPoint(x: 0, y: y))
            bezierPath.addLine(to: CGPoint(x: size.width, y: y))
        }
        
        /// stroke style
        SKColor(white: 0, alpha: 1).setStroke()
        bezierPath.lineWidth = 1
        
        /// draw
        bezierPath.stroke()
    }
    
    /// return a texture that SpriteKit can use
    return SKTexture(image: image)
}
```

We can then use that function to make a `SKSpriteNode`:

```swift
if let gridTexture = generateGridTexture(cellSize: 60, rows: 20, cols: 20) {
    let gridbackground = SKSpriteNode(texture: gridTexture)
    addChild(gridbackground)
}
```

<img src="Screenshots/SpriteKit-CoreGraphics-Gid.png" alt="SpriteKit-CoreGraphics-Gid" style="width:25%;" />

Here is another Core Graphics function that generates a checkerboard texture:

```swift
func generateCheckerboardTexture(cellSize: CGFloat, rows: Int, cols: Int) -> SKTexture? {
    let size = CGSize(width: CGFloat(cols) * cellSize, height: CGFloat(rows) * cellSize)
    
    let renderer = UIGraphicsImageRenderer(size: size)
    let image = renderer.image { ctx in
        let context = ctx.cgContext
        
        /// Draw checkerboard cells
        for row in 0..<rows {
            for col in 0..<cols {
                /// Determine cell color: black for even sum of indexes, white for odd
                let isBlackCell = ((row + col) % 2 == 0)
                context.setFillColor(isBlackCell ? SKColor(white: 0, alpha: 1).cgColor : SKColor(white: 1, alpha: 1).cgColor)
                
                /// Calculate cell frame
                let cellFrame = CGRect(x: CGFloat(col) * cellSize, y: CGFloat(row) * cellSize, width: cellSize, height: cellSize)
                
                /// Fill cell
                context.fill(cellFrame)
            }
        }
    }
    
    return SKTexture(image: image)
}
```

<img src="Screenshots/SpriteKit-CoreGraphics-Checkerboard.png" alt="SpriteKit-CoreGraphics-Checkerboard" style="width:25%;" />

In both cases, we get a sprite node, i.e. a node that draws a bitmap image. This is recommended in SpriteKit, as `SKSpriteNode` offer the best performance, and SpriteKit is a raster renderer anyway (zooming with a camera won't re-render shapes vector-like).

Mind you that Core Graphics is a framework optimized for quality rather than performance. It is not meant to produce images 60 or 120 times per second.

## Core Image filters in SpriteKit

*12 April 2024*

When you apply a Core Image filter in SpriteKit, it's important to understand the difference between a screen-space effect and an object-space or scene-space effect. Filters are always applied through `SKEffectNode`, therefore, the filter's effects will be drawn relative to that node. A node moving inside the effect node will "traverse" the effect applied on the effect node. For example, if you apply a pointillize effect, the dots will "belong" to the effect node, not to the nodes inside it. If you want the dots to "travel" with a moving node, you have to move the effect node itself. At which point, you might as well produce a static texture with Core Image, that you use as an `SKTexture` for a sprite node, which is better for performance.

Screen recordings:

- The first video applies a filter in screen-space: the effect node contains the moving node
- The second video applies the filter in object-space: the effect node is the moving node

<video src="Screenshots/SpriteKit-Filter-Pointillize-Screen.mp4" width="310"></video>

<video src="Screenshots/SpriteKit-Filter-Pointillize-Object.mp4" width="310"></video>

https://github.com/AchrafKassioui/Learning-SpriteKit/assets/1216689/ce77b01b-b241-4c4f-9f7e-7d66c9f7dea9

https://github.com/AchrafKassioui/Learning-SpriteKit/assets/1216689/3eb90542-add2-42b7-bca0-9ea8f9781c6a

*17 March 2024*

SpriteKit has built-in methods using Core Image filters for:

- Applying filters to nodes of type `SKEffectNode`. Out of the box, `SKEffectNode.filter` accepts filters that have a single inputImage parameter and produce a single outputImage parameter.
- Applying filters to scene transitions with `SKTransition`. Out of the box, `SKTransition.init(ciFilter:duration:)` accepts filters that require only two image parameters (inputImage, inputTargetImage) and generate a single image (outputImage).
- Applying filters to textures of type `SKTexture` to produce a new texture. Out of the box, `SKTexture.applying(CIFilter)` accepts filters that require a single inputImage parameter and produce an outputImage parameter.

## Shape nodes

*14 March 2024*

**TL;DR**: Nodes of type `SKShapeNode` can generate shapes with code, providing the ability to draw and edit shapes dynamically during runtime. SpriteKit's ability to generate accurate physics bodies for shape nodes is more limited than it is with sprite nodes. When rendered, a shape node is rasterized, so when it's drawn in a size other than its native size, some filtering is applied (blurring or aliasing).

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

Here are other useful code samples:

```swift
/// change the radius of an existing round shape
let myCircle = SKShapeNode(circleOfRadius: 10)
let newRadius: CGFloat = 10
let newPath = CGPath(ellipseIn: CGRect(x: -newRadius, y: -newRadius, width: newRadius*2, height: newRadius*2), transform: nil)
myCircle.path = newPath
```

```swift
/// create a path, then dynamically assign it to a shape
let path = CGMutablePath()
path.move(to: CGPoint(x: -10, y: 0))
path.addLine(to: CGPoint(x: 10, y: 0))
path.move(to: CGPoint(x: 0, y: 10))
path.addLine(to: CGPoint(x: 0, y: -10))

let shape = SKShapeNode()
shape.path = path
```

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

// same as above, with a different syntax
enumerateChildNodes(withName: "nodeName", using: { node, _ in
	
})

// same as above, with the added stop parameter
// Use `stop.pointee = true` or `stop[0] = true` to stop the enumeration
enumerateChildNodes(withName: "nodeName") {node, stop in
	
}

// code for myNode with name "targetName"
if let myNode = childNode(withName: "//my-node") {
    
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

## Fonts

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

1.

<img src="Screenshots/Xcode add font - 1 Target info.png" alt="Xcode add font - 1 Target info" style="width:100%;" />

2.

<img src="Screenshots/Xcode add font - 2 Fonts provided by application.png" alt="Xcode add font - 2 Fonts provided by application" />

3.

<img src="Screenshots/Xcode add font - 3 Fonts provided by application.png" alt="Xcode add font - 3 Fonts provided by application" />

4.

<img src="Screenshots/Xcode add font - 4 Add font name.png" alt="Xcode add font - 4 Add font name" />

## Centering a node inside another one

*12 March 2024*

For any two nodes, you can center one relative to another with:

```swift
nodeToCenter.position = CGPoint(x: referenceNode.frame.midX, y: referenceNode.frame.midY)
```

There are special cases where a node will be automatically centered. If your node to center has a parent, and if the parent has a property of `anchorNode`, such as `SKScene` and `SKSpriteNode`, then any child node of those with position ``CGPoint(x: 0, y: 0)` are automatically centered relative to their parent node.

## Visualize the frame of a node

*9 March 2024*

The accumulated frame of a node is the smallest straight rectangle that can contain that node. A convex node has a rectangular frame. A rotated square node has a rectangular frame that has expanded to contain the rotated square.

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

You can either call that function one time, or you can call it every frame. The way you call your function every time matters. For example, if your target node is affected by physics, and if you call `visualizeFrame` inside the `update` function, then the bounding box will lag behind your moving physical object, because `update` is called *before* physics are calculated. Therefore, you should call you function inside `didSimulatePhysics` instead:

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

In SpriteKit, you can get a texture from any node, which includes its children. For example, you could define a node using `SKShapeNode`, get the texture of that node, and create a `SKSpriteNode` with the texture.

I use that feature to create better physics bodies for nodes such as `SKShapeNode`:

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

For single touch handling, here is a basic setup:

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

*Update 20 March 2024*

For multi-touch handling, here is basic setup. Note that for multi-touch, a guard statement to ensure there is actually a touch is not necessary, since 1) we are interested in all touches, and 2) the loop over the touches ensures that we only process existing touch points.

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    for touch in touches {
        // process this particular touch point
    }
}

override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
    for touch in touches {
        // process this particular touch point
    }
}

override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    for touch in touches {
        // process this particular touch point        
    }
}

override func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent?) {
    // cleanup the ongoing touch processing
}
```

This how to get the touched nodes:

```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    guard let scene = scene else { return }

    for touch in touches {
        let touchLocation = touch.location(in: scene)
        let touchedNodes = scene.nodes(at: touchLocation)
        for node in touchedNodes {
            // code for every touched node
        }
    }
}
```

Constrain a position within a range:

```swift
override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
    for touch in touches {
        let touchLocation = touch.location(in: self)

        /// Clamp the values
        let lowerBound = -100
        let upperBound = 100
        let allowedYPosition = max(lowerBound, min(touchLocation.y, upperBound))

        myNode.position.y = allowedYPosition
    }
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

Trivia: on iPhone, it seems that the maximum supported amount of simultaneous touch points is 5. Beyond 5 touches, all UITouch objects are canceled.

*Update 20 march 2024*: multitouch is disabled by default on all UIKit views, not just SKView. See [Apple documentation](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_touches_in_your_view):

> In its default configuration, a view receives only the first UITouch object associated with an event, even if more than one finger is touching the view. To receive the additional touches, you must set the view’s isMultipleTouchEnabled property to true.

## SpriteKit text blurriness

*5 December 2023*

SpriteKit nodes of type `SKLabelNode`, aka text, get blurry when the camera is zoomed in. See https://stackoverflow.com/a/72286447/420176

One hack around it is to internally multiply the font size by a scale factor, then scale the node down by the same factor, to get a better rendering when zoomed in.

```Swift
let myText = SKLabelNode()
let textScaleFactor: CGFloat = 5.0
myText.fontSize = 28 * textScaleFactor
myText.name = "myText"
myText.text = "Hello"
myText.fontName = "Impact"
myText.xScale = 1 / textScaleFactor
myText.yScale = 1 / textScaleFactor
addChild(myText)
```

Further testing is required to see how the scaling may affect other behaviors such as physics simulations.

Update: while adding many emojis to the same `SKLabelNode`, the app crashed with an error about Metal's maximum texture size. I believe that given the internal multi-sampling introduced above, and emojis being images, having many of them in the same SpriteKit node eventually exceeds the rendering engine constraints. `SKLabelNode` text length must be limited.

https://en.wikipedia.org/wiki/Spatial_anti-aliasing#Super_sampling_/_full-scene_anti-aliasing

## Setup with SwiftUI

*Updated 16 March 2024*

Minimal boilerplate code to display a SpriteKit scene and preview it inside Xcode 15+ using SwiftUI:

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

#Preview {
    ContentView()
}

// SpriteKit
class MyScene: SKScene {
    
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

Most tutorials and resources available on SpriteKit will use a specific size for the scene, and pass it to the initializer method. How should you size your SpriteKit scene? How does scene size affect performance? Will my nodes' position be constrained by the scene size? If a physics body falls under gravity, will it continue to fall indefinitely?

In reality, scene size is the size of its *visible* portion. The scene itself is infinite. Below are references to better understand the relationship between the scene (infinite) and its presenter (the view, finite).

*28 March 2024*

> A scene’s size defines its visible area. When a scene is first initialized, its size property is configured by the designated initializer. The size of the scene specifies the size of the visible portion of the scene in points. This is only used to specify the visible portion of the scene. Nodes in the tree can be positioned outside of this area; those nodes are still processed by the scene, but are ignored by the renderer.

Source: [SpriteKit Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsAnimation/Conceptual/SpriteKit_PG/Nodes/Nodes.html).

*21 January 2024*

If we command-click on the type `SKScene` in Xcode, we can bring up the header information for the class:

> A scene is infinitely large, but it has a viewport that is the frame through which you present the content of the scene. The passed in size defines the size of this viewport that you use to present the scene.

An older version of the header, [quoted here](https://stackoverflow.com/a/33447352/420176), reads:

> To display different portions of your scene, move the contents relative to the viewport. One way to do that is to create a SKNode to function as a viewport transformation. That node should have all visible contents parented under it.

That version probably predates the introduction of `SKCameraNode`, since a SpriteKit camera does essentially that. As for the relation between the scene size and the viewport size, we can read this in the same header file:

> fill: Scale the SKScene to fill the entire SKView
>
> aspectFill: Scale the SKScene to fill the SKView while preserving the scene's aspect ratio. Some cropping may occur if the view has a different aspect ratio.
>
> aspectFit: Scale the SKScene to fit within the SKView while preserving the scene's aspect ratio. Some letterboxing may occur if the view has a different aspect ratio.
>
> resizeFill: Modify the SKScene's actual size to exactly match the SKView.

So a SpriteKit scene is an infinite canvas by default. The part of the scene that is being drawn and rendered is the view (`SKView`). A scene can either be scaled to fit a view (one of the 4 `scaleMode`), or be drawn through a camera frame that determines which crop of the scene is in view.

Regardless of scene size, objects can be positioned freely without limit. Does positioning objects tens or hundreds of thousands of points away from each other have an impact on memory consumption and performance? I'm not sure yet.

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

## Code samples

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
    // a size for the square
    let boxSize = CGSize(width: 50, height: 50)
    // use the size to create a red sprite
    let physicalBox = SKSpriteNode(color: .red, size: boxSize)
    // use the size to create a physics body
    physicalBox.physicsBody = SKPhysicsBody(rectangleOf: boxSize)
    
    addChild(physicalBox)
}
```

```swift
// Scene with transparent background
import SwiftUI
import SpriteKit

struct transparentSpriteView: View {
    var myScene = SpriteKitScene()
    
    var body: some View {
        SpriteView(
            scene: myScene,
            options: [.allowsTransparency] // Step 1
        )
        .background(.blue)
    }
}

class SpriteKitScene: SKScene {    
    override func didMove(to view: SKView) {
        size = view.bounds.size
        scaleMode = .resizeFill
        backgroundColor = .clear // Step 2
    }
}
```

## Links & Resources

- 📝 [Hanging chains with physics joints](http://www.waveworks.de/howto-make-hanging-chains-sprite-kit-physics-joints/), *accessed 31 March 2024*
- 📐 [SKPhysicsBody Path Generator](http://insyncapp.net/SKPhysicsBodyPathGenerator.html): an old page that generates a CGPath by manually drawing lines over a sprite on the browser. The generated code is in Objective-C with an old syntax, but still interesting to know.
- 🔎 [SpriteKit repositories](https://github.com/topics/spritekit). Search GitHub with tag "spritekit"
- 📝 [SpriteKit From Scratch](https://code.tutsplus.com/series/spritekit-from-scratch--cms-1018), envato tut+
- ⚙️ [Xcode Asset Catalog Generator](https://github.com/artstorm/xcode-asset-catalog-generator), bitbebop.com
- 📝 [Good blog posts on SpriteKit and GameplayKit](https://blog.bitbebop.com/tags/spritekit/), bitbebop.com
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

This file started on 7 July 2023 as a collection of ready-to-use boilerplates called "SpriteKit Cheatsheet". In March 2024, it became a list of timestamped notes covering various SpriteKit-related topics.
