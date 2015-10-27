
### 从 iOS 应用调用自定义 API

若要从 iOS 客户端调用此自定义 API，请使用 `MSClient invokeAPI` 方法。此方法有两个版本，一个版本适用于 JSON 格式的请求，另一个版本适用于任何数据类型：

	/// Invokes a user-defined API of the Mobile Service.  The HTTP request and
	/// response content will be treated as JSON.
	-(void)invokeAPI:(NSString *)APIName
	            body:(id)body
	      HTTPMethod:(NSString *)method
	      parameters:(NSDictionary *)parameters
	         headers:(NSDictionary *)headers
	      completion:(MSAPIBlock)completion;

	/// Invokes a user-defined API of the Mobile Service.  The HTTP request and
	/// response content can be of any media type.
	-(void)invokeAPI:(NSString *)APIName
	            data:(NSData *)data
	      HTTPMethod:(NSString *)method
	      parameters:(NSDictionary *)parameters
	         headers:(NSDictionary *)headers
	      completion:(MSAPIDataBlock)completion;


例如，若要将 JSON 请求发送到名为“sendEmail”的自定义 API，请为请求参数传递 `NSDictionary`：

	NSDictionary *emailHeader = @{ @"to": @"email.com", @"subject" : @"value" };

	[self.client invokeAPI:@"sendEmail"
	     body:emailBody
	     HTTPMethod:@"POST"
	     parameters:emailHeader
	     headers:nil
	     completion:completion ];
		

<!---HONumber=74-->