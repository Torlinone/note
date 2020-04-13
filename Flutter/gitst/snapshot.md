

[stack overflow](https://stackoverflow.com/questions/53646649/how-to-take-screenshot-of-widget-beyond-the-screen-in-flutter)

```dart
 RepaintBoundary(
                      key: _snapKey,
                      child: CounterList(
                        list: box.values.toList(),
                        provider: provider,
                        controller: _snapCtrl,
                      ),
                    );


 Future<void> _captureImage() async {
    ui.PictureRecorder recorder = ui.PictureRecorder();
    final paint = Paint();
    ui.Canvas canvas = ui.Canvas(recorder);

    RenderRepaintBoundary boundary = _snapKey.currentContext.findRenderObject();

    Size size = boundary.size;

    var image = await boundary.toImage();
    ByteData byteData = await image.toByteData(format: ui.ImageByteFormat.png);
    Uint8List pngBytes = byteData.buffer.asUint8List();
    ImageProvider imageProvider = MemoryImage(pngBytes);
    ui.Image uiImage = await _loadImageFromProvider(imageProvider);
    canvas.drawImage(uiImage, Offset.zero, paint);

    _snapCtrl.jumpTo(size.height);
    await Future.delayed(Duration(seconds: 1));

    image = await boundary.toImage();
    byteData = await image.toByteData(format: ui.ImageByteFormat.png);
    pngBytes = byteData.buffer.asUint8List();
    imageProvider = MemoryImage(pngBytes);
    uiImage = await _loadImageFromProvider(imageProvider);
    canvas.drawImage(uiImage, Offset(0, size.height), paint);

    var done = await recorder.endRecording().toImage(size.width.toInt(), size.height.toInt() * 2);
    var data = await done.toByteData(format: ui.ImageByteFormat.png);

    await Navigator.push(
      context,
      MaterialPageRoute(
        builder: (_) => Scaffold(
          body: Image.memory(data.buffer.asUint8List()),
        ),
      ),
    );

    return;
  }

  ///load ui.Image from ImageProvider
  ///[provider] ImageProvider
  Future<ui.Image> _loadImageFromProvider(
    ImageProvider provider, {
    ImageConfiguration config = ImageConfiguration.empty,
  }) async {
    Completer<ui.Image> completer = Completer<ui.Image>();
    ImageStreamListener listener;
    ImageStream stream = provider.resolve(config);
    listener = ImageStreamListener((ImageInfo frame, bool sync) {
      final ui.Image image = frame.image;
      completer.complete(image);
      stream.removeListener(listener);
    });
    stream.addListener(listener);
    return completer.future;
  }
  ```