# Content Security Polic (网页安全政策,缩写 CSP)
CSP是一种由开发者定义的安全性政策性申明，通过CSP所约束的的规责指定可信的内容来源（这里的内容可以指脚本、图片、iframe、fton、style等等可能的远程的资源）。通过CSP协定，让WEB处于一个安全的运行环境中。

CSP 大大增强了网页的安全性。攻击者即使发现了漏洞，也没法注入脚本，除非还控制了一台列入了白名单的可信主机。

## 两种方法可以启用 CSP。
一种是通过 HTTP 头信息的Content-Security-Policy的字段

    Content-Security-Policy: script-src 'self'; object-src 'none';
	style-src cdn.example.org third-party.org; child-src https:

另一种是通过网页的<meta>标签
	<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">


## 资源加载限制指令表
<table border="0" cellpadding="0" cellspacing="0" width="100%">
  <tbody>
    <tr>
      <th>指令</th>
      <th>指令值示例</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>default-src</td>
      <td>'self' cnd.a.com</td>
      <td><p>定义针对所有类型（js、image、css、web font，ajax请求，iframe，多媒体等）资源的默认加载策略，某类型资源如果没有单独定义策略，就使用默认的。</p></td>
    </tr>
    <tr>
      <td>script-src</td>
      <td>'self' js.a.com</td>
      <td>定义针对JavaScript的加载策略。</td>
    </tr>
    <tr>
      <td>style-src</td>
      <td>'self' css.a.com</td>
      <td>定义针对样式的加载策略。</td>
    </tr>
    <tr>
      <td>img-src</td>
      <td>'self' img.a.com</td>
      <td>定义针对图片的加载策略。</td>
    </tr>
    <tr>
      <td>connect-src</td>
      <td>'self'</td>
      <td><p>针对Ajax、WebSocket等请求的加载策略。不允许的情况下，浏览器会模拟一个状态为400的响应。</p></td>
    </tr>
    <tr>
      <td>font-src</td>
      <td>font.a.com</td>
      <td>针对Web Font的加载策略。</td>
    </tr>
    <tr>
      <td>object-src</td>
      <td>'self'</td>
      <td>针对&lt;object&gt;、&lt;embed&gt;或&lt;applet&gt;等标签引入的flash等插件的加载策略。</td>
    </tr>
    <tr>
      <td>media-src</td>
      <td>media.a.com</td>
      <td>针对&lt;audio&gt;或&lt;video&gt;等标签引入的html多媒体的加载策略。</td>
    </tr>
    <tr>
      <td>frame-src</td>
      <td>'self'</td>
      <td>针对frame的加载策略。</td>
    </tr>
    <tr>
      <td>sandbox</td>
      <td>allow-forms</td>
      <td><p>对请求的资源启用sandbox（类似于iframe的sandbox属性）。</p></td>
    </tr>
    <tr>
      <td>report-uri</td>
      <td>/report-uri</td>
      <td><p>告诉浏览器如果请求的资源不被策略允许时，往哪个地址提交日志信息。</p>
        <p>特别的：如果只想让浏览器汇报日志，而不阻止任何内容。可以改用Content-Security-Policy-Report-Only响应头。</p></td>
    </tr>
  </tbody>
</table>

##指令值组合说明
<table border="0" cellpadding="0" cellspacing="0" width="100%">
  <tbody>
    <tr>
      <th>指令值</th>
      <th>指令示例</th>
      <th>说明</th>
    </tr>
    <tr>
      <td>*</td>
      <td>img-src *</td>
      <td>允许任何内容。</td>
    </tr>
    <tr>
      <td>'none'</td>
      <td>img-src 'none'</td>
      <td>不允许任何内容。</td>
    </tr>
    <tr>
      <td>'self'</td>
      <td>img-src 'self'</td>
      <td>允许来自相同来源的内容（相同的协议、域名和端口）。</td>
    </tr>
    <tr>
      <td>data</td>
      <td>img-src data</td>
      <td>允许data:协议（例如base64编码的图片）。</td>
    </tr>
    <tr>
      <td><em>www.a.com</em></td>
      <td>img-src img.a.com</td>
      <td>允许加载指定域名的资源。</td>
    </tr>
    <tr>
      <td><em>*.a.com</em></td>
      <td>img-src *.a.com</td>
      <td>允许加载a.com任何子域的资源。</td>
    </tr>
    <tr>
      <td><em>https://img.com</em></td>
      <td>img-src https://img.com</td>
      <td>允许加载img.com的https资源（协议需匹配）。</td>
    </tr>
    <tr>
      <td>https:</td>
      <td>img-src https:</td>
      <td>允许加载https资源。</td>
    </tr>
    <tr>
      <td>'unsafe-inline'</td>
      <td>script-src 'unsafe-inline'</td>
      <td><p>允许加载inline资源（例如常见的style属性，onclick，inline js和inline css等等）。</p></td>
    </tr>
    <tr>
      <td>'unsafe-eval'</td>
      <td>script-src 'unsafe-eval'</td>
      <td>允许加载动态js代码，例如eval()。</td>
    </tr>
  </tbody>
</table>


##运营商劫持临时解决方案
    <!--脚本加载白名单 【临时解决方案，后续还是要使用https来解决这个问题】--> 
	<meta http-equiv="Content-Security-Policy" content="script-src 'unsafe-inline' 'unsafe-eval' *.sjzhushou.com *.xunlei.com">

##浏览器兼容性以及相关资料
  - [http://caniuse.com/#feat=contentsecuritypolicy ](http://caniuse.com/#feat=contentsecuritypolicy  "contentsecuritypolicy ")
- [http://www.html5rocks.com/en/tutorials/security/content-security-policy/](http://www.html5rocks.com/en/tutorials/security/content-security-policy/ "content-security-policy")
