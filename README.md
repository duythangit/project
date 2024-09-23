# Làm quen với ứng dụng web động đơn giản sử dụng Flutter và Dart

## Mục tiêu

- Hiểu và áp dụng được các khái niệm cơ bản về ứng dụng web động, ứng dụng đa nền tảng
- Sử dụng Flutter Framework để tạo giao diện đơn giản cho 1 ứng dụng
- Sử dụng Dart và thư viện shelf, shelf_router để tạo server đơn giản xử lí các yêu cầu HTTP theo chuẩn RESTful API
- Tích hợp giao diện với logic xử lý phản hồi từ server, thực hiện thao tác gửi dữ liệu từ client lên server thông qua HTTP POST
- Kiểm thử đơn giản với Postman để kiểm tra phản hồi từ server đối với các yêu cầu GET và POST bao gồm cả trường hợp hợp lệ và không hợp lệ

## Cấu trúc thư mục

Để tiện cho việc quản lý và có thể đầy lên Github chúng ta sẽ cài đặt backend và frontend trong cùng 1 thư mục dự án.

plaintext
simple_flutter_project\
|--frontend/ # thư mục chứa mã nguồn Dart và Flutter cho frontend
|--backend/ # thư mục chứa mã nguồn Dart cho backend

## Các bước thực hiện

### Bước 1 : Khởi tạo dự án

1. Tạo thư mục chứa dự án simple_flutter_project
2. Tạo thư mục frontend và backend trong thư mục simple_flutter_project như cấu trúc ở trên
3. Mở ứng dụng VS code và mở thư mục simple_flutter_project

### Bước 2 : Tạo ứng dụng server cho backend với Dart Framewwork

1. Đi đến thư mục backend từ thư mục simple_flutter_project

cd backend

2. Khởi tạo dự án Dart mới cho server

dart create -t server-shelf . --force

_Lưu ý_

1.  Phải cài Flutter
2.  Lệnh dart create -t server-shelf . --force sẽ tạo 1 dự án Dart với mẫu -t, template là server-shelf trong thư mục hiện tại . và tham số --force cho biết sẽ tạo dự án cho dù thư mục gốc đã tồn tại ( mặc định là sẽ tạo mới thư mục )

3.  Thêm các thư viện vào dự án backend nếu cần

- Trong ứng dụng mẫu server-shelf flutter đẫ sử dụng các thư viện shelf và shelf-router trong tệp` pubspec.yaml`
- Có thể xem thư viện khác ở pub.dev

### Bước 3 : Tạo ứng dụng frontend bằng Flutter framework

1. Quay lại thư mục frontend (học ở git đã biết cách quay trở lại rồi )
2. Khởi tạo dự án Flutter mới trong frontend

flutter create -e

_Lưu ý_ Lệnh trên sẽ tạo 1 dự án flutter mới trong thư mục frontend với mẫu là Empty Application hay tham số -e và tham số dấu chấm . cho biết sẽ khởi tạo trong thư mục hiện tại là thư mục frontend

3. Thêm thư viện http vào dự án frontend

flutter pub add http

### Bước 4 : Thiết lập debug cho cả frontend và backend

- Mở tệp frontend/lib/main.dart trước
- Chạy Run and Debug và chọn create a launch. json file để tạo file cấu hình gỡ bỏ (debug)
- Tiến hành gỡ lỗi backend và frontend

_Lưu ý_ Chúng ta để cổng mặc định cho server backend là 8080 và frontend là 8081 khi debug và chúng ta có thể thay đổi được

### Bước 5 : Đẩy dự án mã nguồn lên github và quản lý mã nguồn

- Chọn Source Control ở thanh Side Bar và chọn Publish to Github
- Quản lý mã nguồn bằng cách commit , push (Sync Changes ...), pull, .... từ cửa sổ Source Control

### Bước 6: Phát triển backend và kiểm thử

1. Chỉnh sửa file `backend/bin/server.dart`

- Mở file `server.dart` và chỉnh sửa:

```bash
import 'dart:convert';
import 'dart:io';

import 'package:shelf/shelf.dart';
import 'package:shelf/shelf_io.dart';
import 'package:shelf_router/shelf_router.dart';

// cấu hình các router
final _router = Router(notFoundHandler: _notFoundHandler)
  ..get('/', _rootHandler)
  ..get('/api/v1/check', _checkHandler)
  ..get('/api/v1/echo/<message>', _echoHandler)
  ..post('/api/v1/submit', _submitHandler);

///Header mặc định cho dữ liệu trả về dưới dạng JSON
final _headers = {'Content-Type': 'application/json'};
Response _notFoundHandler(Request req) {
  return Response.notFound('Không tìm thấy đường dẫn "${req.url}" trên server');
}

/// Hàm xử lý các yêu cầu gốc tại đường dẫn '/'
///
/// Trả về 1 phản hồi với thông điệp `Hello, World!` dưới dạng JSON
///
/// reg : Đối tượng yêu cầu từ client
///
/// Trả về : một đối tượng Response với mã trạng thái 200 và nội dung JSON
Response _rootHandler(Request req) {
  /// Constructor ok của Response có statusCode là 200
  return Response.ok(
    json.encode({'message': 'Hello, World!'}),
    headers: _headers,
  );
}

/// Hàm xử lý yêu cầu tại đường dẫn `/api/v1/check`
Response _checkHandler(Request req) {
  return Response.ok(
    json.encode({'message': 'Chào mừng bạn đến với ứng dụng web động'}),
    headers: _headers,
  );
}

Response _echoHandler(Request request) {
  final message = request.params['message'];
  return Response.ok('$message\n');
}

Future<Response> _submitHandler(Request req) async {
  try {
    /// Đọc payload từ request
    final payload = await req.readAsString();
    // Giaỉ mã JSON từ payload
    final data = json.decode(payload);
    // Lấy giá trị 'name' từ data, ép kiểu về String? nếu có
    final name = data['name'] as String?;
    // Kiểm tra nếu 'name' hợp lệ
    if (name != null && name.isNotEmpty) {
      // Tạo phản hồi chào mừng
      final response = {'message': 'Chào mừng $name'};
      return Response.ok(
        json.encode(response),
        headers: _headers,
      );
    } else {
      // Tạo phản hồi yêu cầu cung cấp tên
      final response = {'message': 'Server không nhận được tên của bạn.'};
      return Response.badRequest(
        body: json.encode(response),
        headers: _headers,
      );
    }
  } catch (e) {
    /// Xử lý ngoại lệ khi giải mã JSON
    final response = {
      'message': 'Yêu cầu không hợp lệ. Mã lỗi ${e.toString()}'
    };

    /// Trả về phản hồi với statusCode 400
    return Response.badRequest(
      body: json.encode(response),
      headers: _headers,
    );
  }
}

void main(List<String> args) async {
  // Use any available host or container IP (usually `0.0.0.0`).
  final ip = InternetAddress.anyIPv4;

  // Configure a pipeline that logs requests.
  final handler =
      Pipeline().addMiddleware(logRequests()).addHandler(_router.call);

  // For running in containers, we respect the PORT environment variable.
  final port = int.parse(Platform.environment['PORT'] ?? '8080');
  final server = await serve(handler, ip, port);
  print('Server listening on port ${server.port}');
}
```

2. Debug backend và kiểm thử với Postman

### Bước 7 : Phát triển frontend và tích hợp hệ thống
