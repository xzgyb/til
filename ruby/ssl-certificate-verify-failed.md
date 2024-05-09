# 当调用Net::HTTP#connect时，有时会发生connect_nonblock':certificate verify failed的错误．

对应方式如下:

- 如果直接使用Net::HTTP调用，则  
  `http.verify_mode = OpenSSL::SSL::VERIFY_NONE`

- 如果是通过Faraday调用网络请求，则  
  `Faraday.default_connection_options[:ssl].verify_mode = OpenSSL::SSL::VERIFY_NONE`

但是这种使用`VERIFY_NONE`模式禁止校验，并不推荐．

  