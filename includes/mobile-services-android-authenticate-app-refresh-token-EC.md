在简单的情况下，我们的令牌缓存应该有效，但是，在令牌到期或撤销时，会发生什么情况？该应用未运行时，该令牌可能会到期。这就意味着令牌缓存无效。在应用直接进行的调用或移动服务库进行的调用期间，该应用实际运行时，令牌也可能到期。结果将是 HTTP 状态码 401"未授权"。 

我们需要能够检测到期的令牌，并刷新它。为此，我们从 [Android 客户端库](http://dl.windowsazure.com/androiddocs/)使用 [ServiceFilter](http://dl.windowsazure.cn/androiddocs/com/microsoft/windowsazure/mobileservices/ServiceFilter.html)。

在本节中，您将定义 ServiceFilter，它将检测 HTTP 状态码 401 响应，并触发令牌和令牌缓存的刷新。此外，在身份验证期间，此 ServiceFilter 将阻止其他出站请求，以便这些请求可以使用刷新的令牌。

1. 在 Eclipse 中，打开 ToDoActivity.java 文件并添加以下 import 语句：
 
        import java.util.concurrent.atomic.AtomicBoolean;
		import java.util.concurrent.ExecutionException;

		import com.microsoft.windowsazure.mobileservices.MobileServiceException;
 
2. 将以下成员添加到  `ToDoActivity` 类。 

    	public boolean bAuthenticating = false;
	    public final Object mAuthenticationLock = new Object();

    这些将用于帮助同步用户的身份验证。我们只希望进行一次身份验证。在身份验证过程中，任何调用应等待并使用来自正在进行的身份验证的新令牌。

3. 在 ToDoActivity.java 文件中，将以下方法添加到 ToDoActivity 类，该类将用于在正在进行身份验证时阻止其他线程中的出站调用。

	    /**
    	 * Detects if authentication is in progress and waits for it to complete. 
         * Returns true if authentication was detected as in progress. False otherwise.
    	 */
    	public boolean detectAndWaitForAuthentication()
    	{
    		boolean detected = false;
    		synchronized(mAuthenticationLock)
    		{
    			do
    			{
    				if (bAuthenticating == true)
    					detected = true;
    				try
    				{
    					mAuthenticationLock.wait(1000);
    				}
    				catch(InterruptedException e)
    				{}
    			}
    			while(bAuthenticating == true);
    		}
    		if (bAuthenticating == true)
    			return true;
    		
    		return detected;
    	}
    	

4. 在 ToDoActivity.java 文件中，将以下方法添加到 ToDoActivity 类。此方法将实际触发等待，然后在身份验证完成时更新出站请求中的令牌。 

    	
    	/**
    	 * Waits for authentication to complete then adds or updates the token 
    	 * in the X-ZUMO-AUTH request header.
    	 * 
    	 * @param request
    	 *            The request that receives the updated token.
    	 */
    	private void waitAndUpdateRequestToken(ServiceFilterRequest request)
    	{
    		MobileServiceUser user = null;
    		if (detectAndWaitForAuthentication())
    		{
    			user = mClient.getCurrentUser();
    			if (user != null)
    			{
    				request.removeHeader("X-ZUMO-AUTH");
    				request.addHeader("X-ZUMO-AUTH", user.getAuthenticationToken());
    			}
    		}
    	}


5. 在 ToDoActivity.java 文件中，更新 ToDoActivity 类的  `authenticate` 方法，使它接受一个布尔型参数，以允许强制刷新令牌和令牌缓存。我们还需要在身份验证完成时，通知任何被阻止的线程以便其可以选取新令牌。

	    /**
    	 * Authenticates with the desired login provider. Also caches the token. 
    	 * 
    	 * If a local token cache is detected, the token cache is used instead of an actual 
	     * login unless bRefresh is set to true forcing a refresh.
    	 * 
	     * @param bRefreshCache
    	 *            Indicates whether to force a token refresh. 
	     */
    	private void authenticate(boolean bRefreshCache) {
	        
		    bAuthenticating = true;
		    
		    if (bRefreshCache || !loadUserTokenCache(mClient))
	        {
	            // New login using the provider and update the token cache.
	            mClient.login(MobileServiceAuthenticationProvider.MicrosoftAccount,
	                    new UserAuthenticationCallback() {
	                        @Override
	                        public void onCompleted(MobileServiceUser user,
	                                Exception exception, ServiceFilterResponse response) {
    
	                    		synchronized(mAuthenticationLock)
	                    		{
		                        	if (exception == null) {
		                                cacheUserToken(mClient.getCurrentUser());
                                        createTable();
		                            } else {
		                                createAndShowDialog(exception.getMessage(), "Login Error");
		                            }
		            				bAuthenticating = false;
		            				mAuthenticationLock.notifyAll();
	                    		}
	                        }
	                    });
	        }
		    else
		    {
		    	// Other threads may be blocked waiting to be notified when 
		    	// authentication is complete.
		    	synchronized(mAuthenticationLock)
		    	{
		    		bAuthenticating = false;
		    		mAuthenticationLock.notifyAll();
		    	}
                createTable();
		    }
	    }   



6. 在 ToDoActivity.java 文件中，在 ToDoActivity 类中，为新  `RefreshTokenCacheFilter` 类添加此代码：

		/**
		* The RefreshTokenCacheFilter class filters responses for HTTP status code 401. 
		 * When 401 is encountered, the filter calls the authenticate method on the 
		 * UI thread. Out going requests and retries are blocked during authentication. 
		 * Once authentication is complete, the token cache is updated and 
		 * any blocked request will receive the X-ZUMO-AUTH header added or updated to 
		 * that request.   
		 */
		private class RefreshTokenCacheFilter implements ServiceFilter {
		 
	        AtomicBoolean mAtomicAuthenticatingFlag = new AtomicBoolean();                     
	        
	        @Override
	        public ListenableFuture<ServiceFilterResponse> handleRequest(
	        		final ServiceFilterRequest request, 
	        		final NextServiceFilterCallback nextServiceFilterCallback
	        		)
	        {
	            // In this example, if authentication is already in progress we block the request
	            // until authentication is complete to avoid unnecessary authentications as 
	            // a result of HTTP status code 401. 
	            // If authentication was detected, add the token to the request.
	            waitAndUpdateRequestToken(request);
	 
	            // Send the request down the filter chain
	            // retrying up to 5 times on 401 response codes.
	            ListenableFuture<ServiceFilterResponse> future = null;
	            ServiceFilterResponse response = null;
	            int responseCode = 401;
	            for (int i = 0; (i < 5 ) && (responseCode == 401); i++)
	            {
	                future = nextServiceFilterCallback.onNext(request);
	                try {
	                    response = future.get();
	                    responseCode = response.getStatus().getStatusCode();
	                } catch (InterruptedException e) {
	                   e.printStackTrace();
	                } catch (ExecutionException e) {
	                    if (e.getCause().getClass() == MobileServiceException.class)
	                    {
	                        MobileServiceException mEx = (MobileServiceException) e.getCause();
	                        responseCode = mEx.getResponse().getStatus().getStatusCode();
	                        if (responseCode == 401)
	                        {
	                            // Two simultaneous requests from independent threads could get HTTP status 401. 
	                            // Protecting against that right here so multiple authentication requests are
	                            // not setup to run on the UI thread.
	                            // We only want to authenticate once. Requests should just wait and retry 
	                            // with the new token.
	                            if (mAtomicAuthenticatingFlag.compareAndSet(false, true))                                                                                                      
	                            {
	                                // Authenticate on UI thread
	                                runOnUiThread(new Runnable() {
	                                    @Override
	                                    public void run() {
	                                        // Force a token refresh during authentication.
	                                        authenticate(true);
				// ToDoActivity.mMainActivity.authenticate(true);
	
	                                    }
	                                });
	                            }
	
				                // Wait for authentication to complete then update the token in the request. 
				                waitAndUpdateRequestToken(request);
				                mAtomicAuthenticatingFlag.set(false);                                                  
	                        }
	                    }
	                }
	            }
	            return future;
	        }
		}


    该服务过滤器将检查每个响应以查找 HTTP 状态码 401“未授权”。如果遇到 401，将在 UI 线程上设置用于获取新令牌的新登录请求。其他调用将被阻止，直到完成该登录，或 5 次尝试都已失败。如果获得新令牌，则将使用新令牌重试触发 401 的请求，将使用新令牌重试任何被阻止的调用。

7. 在 ToDoActivity.java 文件中，在 ToDoActivity 类中，为新 `ProgressFilter` 类添加此代码：
		
		/**
		* The ProgressFilter class renders a progress bar on the screen during the time the App is waiting for the response of a previous request.
		* the filter shows the progress bar on the beginning of the request, and hides it when the response arrived.
		*/
		private class ProgressFilter implements ServiceFilter {
			@Override
			public ListenableFuture<ServiceFilterResponse> handleRequest(ServiceFilterRequest request, NextServiceFilterCallback nextServiceFilterCallback) {

				final SettableFuture<ServiceFilterResponse> resultFuture = SettableFuture.create();

				runOnUiThread(new Runnable() {

					@Override
					public void run() {
						if (mProgressBar != null) mProgressBar.setVisibility(ProgressBar.VISIBLE);
					}
				});

				ListenableFuture<ServiceFilterResponse> future = nextServiceFilterCallback.onNext(request);

				Futures.addCallback(future, new FutureCallback<ServiceFilterResponse>() {
					@Override
					public void onFailure(Throwable e) {
						resultFuture.setException(e);
					}

					@Override
					public void onSuccess(ServiceFilterResponse response) {
						runOnUiThread(new Runnable() {

							@Override
							public void run() {
								if (mProgressBar != null) mProgressBar.setVisibility(ProgressBar.GONE);
							}
						});

						resultFuture.set(response);
					}
				});

				return resultFuture;
			}
		}
		
	此筛选器将在请求开始时显示进度条，在响应传达时隐藏进度条。
	
8. 在 ToDoActivity.java 文件中，更新 `onCreate` 方法，如下所示：

		@Override
	    public void onCreate(Bundle savedInstanceState) {
		    super.onCreate(savedInstanceState);
		    
		    setContentView(R.layout.activity_to_do);
		    mProgressBar = (ProgressBar) findViewById(R.id.loadingProgressBar);
        
		    // Initialize the progress bar
		    mProgressBar.setVisibility(ProgressBar.GONE);
        
		    try {
		    	// Create the Mobile Service Client instance, using the provided
		    	// Mobile Service URL and key
		    	mClient = new MobileServiceClient(
		    			"https://<YOUR MOBILE SERVICE>.azure-mobile.cn/",
		    			"<YOUR MOBILE SERVICE KEY>", this)
                           .withFilter(new ProgressFilter())
                           .withFilter(new RefreshTokenCacheFilter());
            
		    	// Authenticate passing false to load the current token cache if available.
		    	authenticate(false);
		        	
		    } catch (MalformedURLException e) {
			    createAndShowDialog(new Exception("Error creating the Mobile Service. " +
			        "Verify the URL"), "Error");
		    }
	    }


       在此代码中，除了 `ProgressFilter` 以外，还使用了 `RefreshTokenCacheFilter`。此外，在 `onCreate` 期间，我们想要加载令牌缓存。因此将 `false` 传递给 `authenticate` 方法。

<!---HONumber=74-->