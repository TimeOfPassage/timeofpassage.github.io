```dart
const int PARTITION_SIZE = 80 * 1024 * 1024;
Future<String> getFileHash(String filePath) async {
  final file = File(filePath);
  final fileLength = file.lengthSync();

  final sFile = await file.open();
  try {
    int x = 0;
    const chunkSize = PARTITION_SIZE;
    while (x < fileLength) {
      final tmpLen = fileLength - x > chunkSize ? chunkSize : fileLength - x;
      print(md5.convert(sFile.readSync(tmpLen)));
      x += tmpLen;
    }
    return "hash.toString()";
  } finally {
    unawaited(sFile.close());
  }
}
```

