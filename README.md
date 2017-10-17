
**Introduction and Statement of Purpose:**

This is basic proof-of-concept web application that allows 23andMe customers to access the various forms of their raw genetic data made available by 23andMe's API.

This project is comprised of two distinct repositories or code bases, a frontend and a backend API:

[Frontend source code repository](https://github.com/mdublin/23andMe-KnockoutJS)

[Proxy API source code repository](https://github.com/mdublin/23andMe-Flask-ProxyAPI)


[**VIEW LIVE DEPLOYMENT**](https://23andmefrontend.com/) (**see note on https below)


As will be explained in more detail below, the frontend and the backend are completely decoupled. In other words, this is not an MVC-type architecture wherein the backend, server-side code is rendering the frontend views. The reason that this project does not just consist of a frontend making AJAX calls directly to 23andMe's own API is due to their API's limitations, specifically the lack of JSONP compatibility and cross origin requesting.

**Screenshot of accession data view, application running on Firefox**:
![Screenshot of accession data view, application running on Firefox](https://bytebucket.org/dublinoverflow/23andmespa-proxyapi/raw/f45178e7fa99c2da6916ca05344190ea335d3a60/accessions_view.png)


**Architecture Overview of the 23andme SPA and Proxy API:**

This application can be classified as an SPA (Single Page Application) in that there is only one actual html file that is loaded into the browser and that page is dynamically updated as the user interacts with the application.

While at certain points there can be multiple http requests being made in the background to retrieve resources, the browser will not need to load/reload page views as the users engages with the application.

**Frontend**

Typical of an SPA, the files that make up the frontend component of the application are merely a collection of HTML, CSS, and Javascript files that are statically served. As the user interacts with the application, AJAX calls are made from the frontend to the backend proxy API which in turn makes requests to 23andMe's own API on behalf of the user.

As mentioned earlier, the frontend is completely decoupled from the backend. So for example, anyone could develop their own frontend application, completely distinct from this current version, that integrates with the proxy API endpoints. In addition, this small collection of static files which comprises the entirety of the frontend application (the total file size of the frontend application is ~1.2 MB) can be served on multiple servers and across a range of different deployments scenarios (server stacks, operating systems, etc).

The frontend is primarily based on the Knockout.js framework. The architecture of the frontend is component-based and uses the Asynchronous Module Definition API, which specifies a mechanism for defining modules such that the module and its dependencies can be asynchronously loaded. This is particularly well suited for the browser environment where synchronous loading of modules incurs performance, usability, debugging, and cross-domain access problems. While there are multiple Javascript, CSS, and HTML files that are used to create the frontend, we use a task runner, Gulp, to compile all of these files together into only a single Javascript, CSS, and HTML file. These files, as well as some additional media, are packaged together into a single directory which is then statically hosted.

**Backend**

The backend proxy API is Python based and uses the Flask web framework to help facilitate the creation of various API endpoint routes. The API handles a range of user requests and queries that require interacting with 23andMe's own API for data retrieval as well as user authentication.

While the proxy API does temporarily access user authentication data from 23andMe via OAuth 2.0, it only uses this sensitive data to then create its own JWT (JSON Web Token) for the user session, which is embedded on an httponly cookie that is provided to the frontend. The point is that sensitive 23andMe user account data, including any 23andMe access tokens and account identifiers, are never actually exposed to the frontend. Additionally, in its currently version, the backend API is basically stateless in that there is no database where user account and genetic is stored.  

**Production Implementation:**

The frontend is hosted entirely using an AWS s3 bucket, with a domain registered with AWS Route53 (a DNS provider and server) and another AWS service for load balancing called Cloudfront.

The backend API is hosted on a vanilla AWS EC2 Ubuntu Linux instance using Nginx as a reserve proxy server with Gunicorn and Supervisor for fault-tolerance (see the proxy API README for more information on server setup specifics).

The total cost of this implementation was $12 (for the domain, all other services are available for free).

**Screenshot of variants data view, application running on Firefox**:
![Screenshot of variants data view, application running on Firefox](https://bytebucket.org/dublinoverflow/23andmespa-proxyapi/raw/4e24562a1d84616e30cbeae340b09e8857c92a38/variants_view.png)

**Note: In the interest of maintaining some basic security, the live deployment is only available via https. However, the SSL/TLS certificate that is enabling https is the default CloudFront certificate provided by AWS, it is not from a CA that most browsers will automatically recognize. So, you will be warned and prompted by your browser to manually OK the certificate, but it is definitely SSL/TLS-enabled (I just didn't want to pay for a hundred dollar SSL/TLS certificate):
![](https://bytebucket.org/dublinoverflow/23andmespa-proxyapi/raw/02b296e9a3602d52b110f4d02ae98d54ceb03c22/TLS_proof.png)
