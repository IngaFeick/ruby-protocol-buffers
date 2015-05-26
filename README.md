# Ruby Protocol Buffers

[![Build Status](https://travis-ci.org/codekitchen/ruby-protocol-buffers.png?branch=master)](https://travis-ci.org/codekitchen/ruby-protocol-buffers)

Protocol Buffers are a way of encoding structured data in an efficient yet
extensible format. Google uses Protocol Buffers for almost all of its
internal RPC protocols and file formats.

This library has two components: a compiler to turn `.proto` definitions
into Ruby modules (extension `.pb.rb`), and a runtime to use protocol
buffers defined by these modules.

The compiler relies on Google's C++ based compiler (`protoc`) for much of
the heavy lifting -- this has huge advantages in ensuring compatibility and
correctness. If you don't need cross-language interoperability you can
create Message classes directly in ruby, in which case `protoc` is not
needed. See "Writing Message Classes Directly" below.

This library is heavily optimized for encoding and decoding speed.

Because this is a tool for generating code, the RDoc documentation is a bit
unusual. See the text in the ProtocolBuffers::Message class for details on
what code is generated.

## Installation

    $ gem install ruby-protocol-buffers

If you want to compile .proto files to ruby, you'll need `protoc` version >=
2.2 (the Google Protocol Buffer compiler) installed in the environment where
you will be compiling them.  You do not need `protoc` installed to use the
generated `.pb.rb` files.

For greater performance, consider installing the varint gem as well.  This
optional gem builds a small C extension to make parsing protocol buffers
faster. If your application uses a Gemfile, add varint to the Gemfile
alongside ruby-protocol-buffers.

## Example

Given the file test.proto:

```protobuf
package Test;

message MyMessage {
  optional string myField = 1;
}
```

Compile it to ruby using the command:

    $ ruby-protoc test.proto

Then it can be used from ruby code:

```ruby
require 'test.pb'
msg = Test::MyMessage.new(:myField => 'zomgkittenz')
open("test_msg", "wb") do |f|
  msg.serialize(f)
end
encoded = msg.serialize_to_string # or msg.to_s
Test::MyMessage.parse(encoded) == msg # true
```

## Writing Message Classes Directly

Protocol Buffer definitions are often shared between applications written in
different programming languages, and so are normally defined in .proto files
and translated to ruby using the `ruby-protoc` binary.

However, it's quite simple to write `ProtocolBuffers::Message` classes
directly when a .proto file isn't needed.

```ruby
require 'protocol_buffers'

class User < ProtocolBuffers::Message
  required :string, :name, 1
  required :string, :email, 2
  optional :int32, :logins, 3
end

class Group < ProtocolBuffers::Message
  repeated User, :users, 1
  repeated Group, :subgroups, 2
  
  module GroupType
    include ProtocolBuffers::Enum
    Study = 1
    Play = 2
  end

  optional GroupType, :group_type, 3
end
```

This code is essentially equivalent to the code `ruby-protoc` will generate
if given this .proto file:

```protobuf
message User {
  required string name = 1;
  required string email = 2;
  optional int32 logins = 3;
}

message Group {
  repeated User users = 1;
  repeated Group subgroups = 2;

  enum GroupType {
    Study = 1;
    Play = 2;
  }

  optional GroupType group_type = 3;
}

```

Using a hand-written Message subclass is the same as using a Message class
generated by `ruby-protoc`.

```ruby
group = Group.new(:group_type => Group::GroupType::Play)
group.users << User.new(:name => 'test user', :email => 'test@example.com')
open("group1.test", "wb") do |f|
  group.serialize(f)
end
```

## Features

### Supported Features

* messages, enums, field types, all basic protobuf features
* packages
* imports
* nested types
* passing on unknown fields when re-serializing a message
* groups
* RPC stubbing
* formatting to and parsing from text format

### Currently Unsupported Features

* extensions
* packed option (could be useful)
* accessing custom options

## Protocol Buffers v3

This library supports the Protocol Buffers v2 specification. Google is developing official Ruby support for Protobuf v3, the alpha gem is currently available at https://rubygems.org/gems/google-protobuf but documentation is sparse.

## Authors

Brian Palmer (http://github.com/codekitchen)

## Source

https://github.com/codekitchen/ruby-protocol-buffers

## License

See the LICENSE file included with the distribution for licensing and
copyright details.
