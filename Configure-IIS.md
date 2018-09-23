If you run tusdotnet on IIS you will need to configure the Web.config file to allow large requests. 
If this is not done IIS will buffer the entire request and then return a 404. 

The default value is 4 MB.
Roughly 2 GB (2147483647 bytes) seems to be the max value before IIS once again returns a 404. 

Note that you can still send bigger files using tusdotnet by specifying a `chunkSize` in the client that is smaller than 2147483647 bytes.

# OWIN

You need to add two things:
* maxRequestLenght to system.web/httpRuntime (expressed in KB)
* maxAllowedContentLength to system.webServer/security/requestFiltering/requestLimits (expressed in bytes)

```
<system.web>
    <httpRuntime targetFramework="4.5.2" maxRequestLength="2097151" />
  </system.web>
...
<system.webServer>
	<security>
		<requestFiltering>
			<requestLimits maxAllowedContentLength="2147483647"></requestLimits>
		</requestFiltering>
	</security>
</system.webServer>

```

# ASP.NET Core

You only need to add maxAllowedContentLength to system.webServer/security/requestFiltering/requestLimits (expressed in bytes)

```
<system.webServer>
	<security>
		<requestFiltering>
			<requestLimits maxAllowedContentLength="2147483647"></requestLimits>
		</requestFiltering>
	</security>
</system.webServer>
```

