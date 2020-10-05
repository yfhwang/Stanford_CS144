# lab0

## 环境配置

vm镜像文件在vmware5.2版本下运行正常，6.1版本会报错

```bash
ssh -p 2222 cs144@localhost

#usrname:
cs144

#passwd:
cs144_2020
```

编译使用的代码

```bash
cd build
cmake ..
make format #to normalize the coding style
make #to make sure the code compiles
#make -j2
make check_lab0 #to make sure the automated tests pass
```



## C++编程注意

- Never use malloc() or free()； never use new or delete.
- Essentially never use raw pointers (*), and use "smart" pointers (unique ptr or shared ptr) only when necessary. (You will not need to use these in CS144.)
- Avoid templates, threads, locks, and virtual functions. (You will not need to use these in CS144.)
- Avoid C-style strings (char *str) or string functions (strlen(), strcpy()). These are pretty error-prone. Use a std::string instead.
- Never use C-style casts (e.g., (FILE *)x). Use a C++ static cast if you have to (you generally will not need this in CS144).
- Prefer passing function arguments by const reference (e.g.: const Address & address).
- Make every variable const unless it needs to be mutated.
- Make every method const unless it needs to mutate the object.
- Avoid global variables, and give every variable the smallest scope possible.
- Make frquent small commits as you work, and use commit messages that identify what changed and why.

### 参考文献

Pay particular attention to the documentation for the [FileDescriptor](https://cs144.github.io/doc/lab0/class_file_descriptor.html), [Socket](https://cs144.github.io/doc/lab0/class_socket.html), [TCPSocket](https://cs144.github.io/doc/lab0/class_t_c_p_socket.html), and [Address](https://cs144.github.io/doc/lab0/class_address.html) classes. (Note that a Socket is a type of FileDescriptor, and a TCPSocket is a type of Socket.)

## 3.4 编写webget

```
提交文档位置：
/home/cs144/sponge/writeups
编写代码位置：
/home/cs144/sponge/apps/webget.cc
```

### 目标

使get_URL函数实现：

1. 连接http服务，服务器名称为 &host
2. 向服务器请求URL地址，地址为 &URL
3. 把server传送的所有内容输出，直到"eof"

### 代码

```c++
void get_URL(const string &host, const string &path) {
    // Your code here.
    TCPSocket sock1;
    sock1.connect(Address(host, "http"));
    string request1 = "GET " + path + " HTTP/1.1\r\n"
                    + "Host: " + host + "\r\n"
                    + "Connection: close\r\n\r\n";
    sock1.write(request1);
    sock1.shutdown(SHUT_WR);

    while(!sock1.eof()) {
        cout << sock1.read();
    }
    sock1.close();
    // You will need to connect to the "http" service on
    // the computer whose name is in the "host" string,
    // then request the URL path given in the "path" string.

    // Then you'll need to print out everything the server sends back,
    // (not just one call to read() -- everything) until you reach
    // the "eof" (end of file).

    cerr << "Function called: get_URL(" << host << ", " << path << ").\n";
    cerr << "Warning: get_URL() has not been implemented yet.\n";
}
```

### 解释

第7行：告诉服务器回应完之后可以断开连接

第9行：客户端发完请求之后就不再继续发送请求，因此断开输出流(SHUT_WR)

（不添加 1.2s，添加1.2s）

## 4 基于内存的可靠字节流

编写一个字节流类，实现一系列接口

代码严重抄袭

```c++
//byte_stream.hh

class ByteStream {
  private:
    // Your code here -- add private members as necessary.
    const size_t _capacity;  //管道容量
    size_t bytes_in_pipe;    //管道中的字符数 bytes_pipe.size()
    size_t bytes_read_total; 
    size_t bytes_write_total;
    bool _ended;
    std::deque<char> bytes_pipe;
}
```



```c++
//byte_stream.cc

ByteStream::ByteStream(const size_t capacity): _capacity(capacity),bytes_in_pipe(0),bytes_read_total(0),bytes_write_total(0),_ended(false),bytes_pipe() {}

//读取在pipe中的字符数
size_t ByteStream::buffer_size() const { return bytes_in_pipe; }
//读取当前pipe是否为空
bool ByteStream::buffer_empty() const { return buffer_size() == 0; }
//读取当前pipe的容量
size_t ByteStream::remaining_capacity() const { return _capacity - buffer_size(); }


//设置输入结束标识符
void ByteStream::end_input() {_ended = true;} 
//读取输入是否结束
bool ByteStream::input_ended() const { return _ended; }
//读取是否eof
bool ByteStream::eof() const { return buffer_empty() && input_ended(); }


//读取一共输出的字符数
size_t ByteStream::bytes_written() const { return bytes_write_total; }
//读取一共写入的字符数
size_t ByteStream::bytes_read() const { return bytes_read_total; }


//从&data向pipe中写入字符
//写入字符的数量取决于pip余量和data长度
size_t ByteStream::write(const string &data) {
    size_t min_size = remaining_capacity() < data.size()? remaining_capacity() : data.size();
    for(size_t i = 0; i < min_size; i++) {
        bytes_pipe.push_back(data[i]);
    }
    bytes_write_total += min_size;
    bytes_in_pipe += min_size;

    return min_size;
}

//取pipe头部的len个字符输出
string ByteStream::peek_output(const size_t len) const {
    size_t min_size = len < buffer_size()? len : buffer_size();
    return string(bytes_pipe.begin(), bytes_pipe.begin() + min_size);
}

//弹出pipe头部的len个字符
void ByteStream::pop_output(const size_t len) {
    size_t min_size = len < buffer_size()? len : buffer_size();
    for(size_t i = 0; i < min_size; i++) {
        bytes_pipe.pop_front();
    }
    bytes_in_pipe -= min_size;
    bytes_read_total += min_size;

}

//read的操作即：先peek，后pop
std::string ByteStream::read(const size_t len) {
    std::string res = peek_output(len);
    pop_output(len);
    return res;
}
```















