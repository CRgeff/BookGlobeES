dependencies:
  flutter:
    sdk: flutter
  file_picker: ^6.1.1
  pdf_text: ^2.0.0
  google_mlkit_translation: ^0.7.0
  translator: ^0.1.7
import 'package:flutter/material.dart';
import 'package:file_picker/file_picker.dart';
import 'dual_reader_page.dart';
import 'pdf_service.dart';

void main() => runApp(BookGlobeApp());

class BookGlobeApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BookGlobe',
      theme: ThemeData(primarySwatch: Colors.indigo),
      home: HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  void _pickAndTranslatePDF(BuildContext context) async {
    final result = await FilePicker.platform.pickFiles(type: FileType.custom, allowedExtensions: ['pdf']);
    if (result != null && result.files.single.path != null) {
      final path = result.files.single.path!;
      final originalText = await PDFService.extractText(path);
      Navigator.push(
        context,
        MaterialPageRoute(builder: (_) => DualReaderPage(originalText: originalText)),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('BookGlobe')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => _pickAndTranslatePDF(context),
          child: Text('Importar Livro (PDF)'),
        ),
      ),
    );
  }
}
import 'package:pdf_text/pdf_text.dart';

class PDFService {
  static Future<String> extractText(String filePath) async {
    PDFDoc doc = await PDFDoc.fromPath(filePath);
    return await doc.text;
  }
}
import 'package:translator/translator.dart';

class TranslatorService {
  static final translator = GoogleTranslator();

  static Future<String> translateText(String text, {String to = 'pt'}) async {
    final translation = await translator.translate(text, to: to);
    return translation.text;
  }
}
import 'package:flutter/material.dart';
import 'translator_service.dart';

class DualReaderPage extends StatefulWidget {
  final String originalText;

  DualReaderPage({required this.originalText});

  @override
  _DualReaderPageState createState() => _DualReaderPageState();
}

class _DualReaderPageState extends State<DualReaderPage> {
  String translatedText = "Traduzindo...";
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    _translate();
  }

  void _translate() async {
    final translated = await TranslatorService.translateText(widget.originalText);
    setState(() {
      translatedText = translated;
      isLoading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Leitura em Tempo Real")),
      body: isLoading
          ? Center(child: CircularProgressIndicator())
          : Row(
              children: [
                Expanded(
                  child: Padding(
                    padding: const EdgeInsets.all(12),
                    child: SingleChildScrollView(child: Text(widget.originalText)),
                  ),
                ),
                VerticalDivider(),
                Expanded(
                  child: Padding(
                    padding: const EdgeInsets.all(12),
                    child: SingleChildScrollView(child: Text(translatedText)),
                  ),
                ),
              ],
            ),
    );
  }
}

