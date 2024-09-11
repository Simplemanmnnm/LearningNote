# nodeJS

- 安装依赖

  ```shell
  # 检查package.json文件，下载并安装所有列在dependencies字段中的依赖
  npm install
  ```

- 安装失败

  ```
  npm ERR! code SELF_SIGNED_CERT_IN_CHAIN
  npm ERR! errno SELF_SIGNED_CERT_IN_CHAIN
  npm ERR! request to https://registry.npmmirror.com/express failed, reason: self-signed certificate in certificate chain
  
  禁用严格的 SSL 模式：运行以下命令，取消 npm 的 HTTPS 认证：
  npm set strict-ssl false
  ```
