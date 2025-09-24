// GitHub ready Flutter Web + Firebase template for Smart Curriculum App

// main.dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/login_screen.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Smart Curriculum App',
      theme: ThemeData(primarySwatch: Colors.purple),
      home: LoginScreen(),
    );
  }
}


// screens/login_screen.dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'attendance_screen.dart';
import 'teacher_dashboard.dart';

class LoginScreen extends StatefulWidget {
  @override
  _LoginScreenState createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  final FirebaseAuth _auth = FirebaseAuth.instance;

  void login() async {
    try {
      UserCredential userCredential = await _auth.signInWithEmailAndPassword(
        email: _emailController.text.trim(),
        password: _passwordController.text.trim(),
      );
      final user = userCredential.user;

      final snapshot = await FirebaseFirestore.instance
          .collection('users')
          .doc(user!.uid)
          .get();
      final role = snapshot['role'];

      if (role == 'teacher') {
        Navigator.pushReplacement(
            context, MaterialPageRoute(builder: (_) => TeacherDashboard()));
      } else {
        Navigator.pushReplacement(
            context, MaterialPageRoute(builder: (_) => AttendanceScreen()));
      }
    } catch (e) {
      ScaffoldMessenger.of(context)
          .showSnackBar(SnackBar(content: Text('Login Failed')));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Container(
          width: 400,
          padding: EdgeInsets.all(20),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(controller: _emailController, decoration: InputDecoration(labelText: 'Email')),
              SizedBox(height: 10),
              TextField(controller: _passwordController, obscureText: true, decoration: InputDecoration(labelText: 'Password')),
              SizedBox(height: 20),
              ElevatedButton(onPressed: login, child: Text('Login')),
            ],
          ),
        ),
      ),
    );
  }
}


// screens/attendance_screen.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class AttendanceScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final String studentId = 'STUDENT_UID'; // replace with logged in student's UID

    return Scaffold(
      appBar: AppBar(title: Text('Your Attendance')),
      body: StreamBuilder(
        stream: FirebaseFirestore.instance
            .collection('attendance')
            .where('studentId', isEqualTo: studentId)
            .snapshots(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          final docs = snapshot.data!.docs;
          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, index) {
              final data = docs[index];
              return ListTile(
                title: Text("Date: ${data['date'] ?? ''}"),
                subtitle: Text("Status: ${data['status'] ?? ''}"),
              );
            },
          );
        },
      ),
    );
  }
}


// screens/teacher_dashboard.dart
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class TeacherDashboard extends StatelessWidget {
  final _subjectController = TextEditingController();
  final _topicController = TextEditingController();
  final _contentController = TextEditingController();

  void addCurriculum() {
    FirebaseFirestore.instance.collection('curriculum').add({
      'subject': _subjectController.text,
      'topic': _topicController.text,
      'contentUrl': _contentController.text,
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Teacher Dashboard')),
      body: Padding(
        padding: EdgeInsets.all(20),
        child: Column(
          children: [
            TextField(controller: _subjectController, decoration: InputDecoration(labelText: 'Subject')),
            TextField(controller: _topicController, decoration: InputDecoration(labelText: 'Topic')),
            TextField(controller: _contentController, decoration: InputDecoration(labelText: 'Content URL')),
            SizedBox(height: 20),
            ElevatedButton(onPressed: addCurriculum, child: Text('Add Curriculum')),
          ],
        ),
      ),
    );
  }
}
