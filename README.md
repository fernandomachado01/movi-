mobilidade_app/
│
├── pubspec.yaml
├── README.md
└── lib/
    ├── main.dart
    └── src/
        ├── app.dart
        ├── models/
        │   └── trip.dart
        ├── services/
        │   ├── api_client.dart
        │   └── socket_service.dart
        ├── utils/
        │   └── location_utils.dart
        └── features/
            ├── splash/
            │   └── splash_page.dart
            ├── auth/
            │   └── otp_page.dart
            ├── home/
            │   └── home_page.dart
            └── driver/
                └── driver_page.dart

name: mobilidade_app
description: MVP mobility app (car + moto) - skeleton
publish_to: 'none'
version: 0.1.0

environment:
  sdk: '>=2.17.0 <3.0.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  http: ^0.13.5
  flutter_riverpod: ^2.0.2
  google_maps_flutter: ^2.2.2
  geolocator: ^9.0.2
  flutter_dotenv: ^5.0.2
  flutter_secure_storage: ^7.0.1
  socket_io_client: ^2.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
# Mobilidade App 🚖

Urban mobility app skeleton (Car + Moto) built with Flutter.

## Features (MVP)
- Splash screen
- OTP login (mock)
- Rider Home (map + request ride)
- Driver Home (go online, see trips)
- API + Socket service placeholders
- Location utilities

## Setup
1. Install Flutter SDK
2. Run `flutter pub get`
3. Add your **Google Maps API key** to Android & iOS
4. Start app: `flutter run`
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'src/app.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const ProviderScope(child: MyApp()));
}
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'features/splash/splash_page.dart';

class MyApp extends ConsumerWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return MaterialApp(
      title: 'Mobilidade',
      theme: ThemeData(primarySwatch: Colors.indigo),
      home: const SplashPage(),
    );
  }
}
import 'dart:async';
import 'package:flutter/material.dart';
import '../auth/otp_page.dart';

class SplashPage extends StatefulWidget {
  const SplashPage({Key? key}) : super(key: key);

  @override
  State<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends State<SplashPage> {
  @override
  void initState() {
    super.initState();
    Timer(const Duration(seconds: 2), () {
      Navigator.of(context).pushReplacement(MaterialPageRoute(builder: (_) => const OtpPage()));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(mainAxisSize: MainAxisSize.min, children: const [
          Icon(Icons.directions_car, size: 96),
          SizedBox(height: 16),
          Text('Mobilidade', style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
        ]),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import '../home/home_page.dart';

class OtpPage extends StatefulWidget {
  const OtpPage({Key? key}) : super(key: key);

  @override
  State<OtpPage> createState() => _OtpPageState();
}

class _OtpPageState extends State<OtpPage> {
  final TextEditingController _phone = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login / OTP')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(children: [
          TextField(controller: _phone, decoration: const InputDecoration(labelText: 'Phone (with country code)')),
          const SizedBox(height: 12),
          ElevatedButton(onPressed: () {
            // TODO: call API to send OTP
            Navigator.of(context).push(MaterialPageRoute(builder: (_) => const HomePage()));
          }, child: const Text('Send OTP'))
        ]),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  GoogleMapController? _controller;
  static const _initialCamera = CameraPosition(target: LatLng(-23.5505, -46.6333), zoom: 14);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Request Ride')),
      body: Stack(children: [
        GoogleMap(initialCameraPosition: _initialCamera, onMapCreated: (c) => _controller = c),
        Align(
          alignment: Alignment.bottomCenter,
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: ElevatedButton(
              onPressed: () {
                // TODO: open request ride sheet
              },
              child: const SizedBox(width: double.infinity, child: Center(child: Text('Request Now'))),
            ),
          ),
        )
      ]),
    );
  }
}
import 'package:flutter/material.dart';

class DriverPage extends StatelessWidget {
  const DriverPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Driver')),
      body: Center(child: Column(mainAxisSize: MainAxisSize.min, children: [
        ElevatedButton(onPressed: () {}, child: const Text('Go Online')),
        const SizedBox(height: 12),
        ElevatedButton(onPressed: () {}, child: const Text('Trips')),
      ])),
    );
  }
}
class Trip {
  final String id;
  final double originLat;
  final double originLng;

  Trip({required this.id, required this.originLat, required this.originLng});
}
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiClient {
  final String baseUrl;
  ApiClient(this.baseUrl);

  Future<Map<String, dynamic>> post(String path, Map<String, dynamic> body) async {
    final res = await http.post(Uri.parse('$baseUrl$path'),
        body: jsonEncode(body), headers: {'Content-Type': 'application/json'});
    return jsonDecode(res.body);
  }
}
import 'package:socket_io_client/socket_io_client.dart' as IO;

class SocketService {
  IO.Socket? socket;

  void connect(String url, String token) {
    socket = IO.io(url, IO.OptionBuilder().setTransports(['websocket']).setQuery({'token': token}).build());
    socket?.onConnect((_) => print('socket connected'));
  }

  void disconnect() {
    socket?.disconnect();
  }
}
import 'package:geolocator/geolocator.dart';

class LocationUtils {
  static Future<Position> getCurrentPosition() async {
    return await Geolocator.getCurrentPosition(desiredAccuracy: LocationAccuracy.high);
  }
}
