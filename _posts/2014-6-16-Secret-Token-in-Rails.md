---
layout: post
title: "Secret Token in Rails"
date: 2014-6-16 11:40:33
---
记得以前在读`Ruby on Rails Tutorial`的时候，里面提到了下面的内容:

> Rails 使用安全权标来加密会话。我们要把硬编码的字符串改为动态生成的。并且修改 .gitignore 文件，不把 .secret 纳入仓库

然后有下面的一段代码，`config/initializers/secret_token.rb`

```ruby
require 'securerandom'

def secure_token
  token_file = Rails.root.join('secret')

  if File.exist?(token_file)
    File.read(token_file).chomp
  else
    token = SecureRandom.hex(64)
    File.write(token_file, token)
    token
  end
end

SampleApp::Application.config.secret_key_base = secure_token
```

当时读的时候对这段代码倒是完全能理解，但是对于`secure_token`在 Rails 中的用途则是一无所知，昨天突然想到了这个然后查阅了一些资料之后有了一定的理解。

> Your secret_token is used for verifying the integrity of you app's session cookies

这时官方文档中对`secret_token`的解释，看来它是用来验证我们的会话 cookies 的合法性的，那么具体的一个过程是怎么样的呢？
首先我们来看看我们一般访问的 Rails 的网站中`session cookie`的样子

```ruby
_MyApp_session=BAh7B0kiD3Nlc3Npb25faWQGOgZFRkkiJTcyZTAwMmRjZTg2NTBiZmI0M2UwZmY0MjEyNGJjODBhBjsAVEkiEF9jc3JmX3Rva2VuBjsARkkiMWhmYTBKSGQwYVQxRlhnTFZWK2FEZEVhbEtLbDBMSitoVEo5YU4zR2dxM3M9BjsARg%3D%3D--dc40a55cd52fe32bb3b84ae0608956dfb5824689
```

`=`后面的部分就是`cookie value`，它被`--`分为了两部分，__第一部分__是一个`Base64`的字符串，它的原始值我们可以在 controllers 中获取到，比如我的最开始的就是这样

```ruby
{"_csrf_token"=>"d09DmGvifqQQOL10CRuCmkmjcfKqFRSYiRN2gMvGDsI="}
```
我们还可以再忘里面添加内容，比如`session['foo'] = 'bar'`之类。__第二部分__也就是签名部分，它用于确保我们的 session 的合法性。

### Session hash
因为目前我们得到的还是一个 hash，所以我们还需要对它进一步处理。

```ruby
session_hash = {"_csrf_token"=>"d09DmGvifqQQOL10CRuCmkmjcfKqFRSYiRN2gMvGDsI="}
# Serialize hash
marshal_dump = Marshal.dump(session_hash)
# => "\x04\b{\x06I\"\x10_csrf_token\x06:\x06EFI\"1d09DmGvifqQQOL10CRuCmkmjcfKqFRSYiRN2gMvGDsI=\x06;\x00F"

# Base64 encode this dump
unescaped_cookie_value = Base64.encode(marshal_dump)
# => "BAh7BkkiEF9jc3JmX3Rva2VuBjoGRUZJIjFkMDlEbUd2aWZxUVFPTDEwQ1J1\nQ21rbWpjZktxRlJTWWlSTjJnTXZHRHNJPQY7AEY=\n"

# Escape line breaks & troublesome characters
escaped_cookie_value = CGI.escape(unescaped_cookie_value).gsub("%0A", "")
# => "BAh7BkkiEF9jc3JmX3Rva2VuBjoGRUZJIjFkMDlEbUd2aWZxUVFPTDEwQ1J1Q21rbWpjZktxRlJTWWlSTjJnTXZHRHNJPQY7AEY%3D"
```

至此，我们完成了 session cookie 的第一部分。

#### Signature
第二部分也是最关键的部分，由于`cookie`是存在于客户端的，因此客户端可以传来任意的字符串，而这也为一些非法攻击埋下了隐患。这时就是`secret_token`派上用场的时候了。第二部分其实就是通过对`secret_token`与第一部分生成的`escape_cookie_value`进行哈希生成的一个签名的字符串。

```ruby
# Calculate the signature using the HMAC digest of the secret_token and the escaped cookie value. Replace %3D with equal sign
cookie_signature = OpenSSL::HMAC.hexdigest(OpenSSL::Digest::SHA1.new, secret_token, escaped_cookie_value.gsub("%3D", "="))
```

这样即使攻击者伪造篡改用户的`cookie`，但是由于由于无法获得`secret_token`，所以也就无法获得正确的签名因而导致 cookie 无效。这有效的确保了 Rails 程序的安全性。

而在 Rails 4.1 之前的版本中，`secret_token`都是直接写在`config/initializers/secret_token.rb`文件中的。而这个文件默认是会加载到 Git 的仓库中，所以一旦被网络上不怀好意的人看到了，将会造成很不好的影响。因此，`Ruby on Rails tutorial`书中建议`secret_token`采用动态生成的方式，将它单独写到`.secret`的文件中，并且纳入到`.gitignore`文件中，这样就能有效的对`secret_token`进行保密。

在 Rails 4.1 中，新加入了`config/secrets.yml`文件，这个文件按通常 Rails 程序的配置方式分为`development``test``production`，前两个环境中的`secret_token`都是直接写死的，对于开发测试模式来说这也是完全合理的。而`production`模式则是这样：

```yaml
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

也就是把`secret_token`直接加入到了环境变量中，当我们在部署 Rails 程序时直接输入相应的`key\value`。

最后记得，`secret_token`一定要长～～～要长～～～，我们也可以直接用命令`rake secret`来替我们生成。
