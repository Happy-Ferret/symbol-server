# Pepto Symbol

Pepto Symbol lets you use [S3](http://aws.amazon.com/s3/) as a [Windows symbol server](http://msdn.microsoft.com/en-us/library/windows/desktop/ms680693%28v=vs.85%29.aspx) for debugging Windows-based applications.

### Wait, what's a symbol server?

"Symbol files" (aka PDB files) contain information about your software that a debugger can use to show you backtraces, local variables, etc. They are generated by the compiler when you build your software. Debugging is a nightmare without symbols. Really.

A "symbol server" hosts symbol files for Windows-based software. Debuggers use symbol servers to obtain symbol files for the software you're debugging automatically. John Robbins has written [a great article explaining all of this](http://www.wintellect.com/CS/blogs/jrobbins/archive/2009/05/11/pdb-files-what-every-developer-must-know.aspx) and more on his blog. Symbol servers can serve symbol files over HTTP or a Windows file share.

### That doesn't sound so hard. Why do I need this?

`symstore.exe`, which creates the directory structure that the symbol server serves, and `symsrv.dll`, which implements the logic for retrieving symbols from a symbol server, operate case-insensitively (like much Windows software). This means that `symsrv.dll` might ask for symbols using a URL that uses uppercase where `symstore.exe` used lowercase, or vice versa.

Many web servers treat URLs case-insensitively. S3 does not. So one job of Pepto Symbol is to provide case-insensitive access to symbol files stored on S3.

In addition, if you try to access a file that doesn't exist on S3, you will get a [403 Forbidden](http://en.wikipedia.org/wiki/HTTP_403) error, rather than [404 Not Found](http://en.wikipedia.org/wiki/HTTP_404). When `symsrv.dll` gets a 403 error from a symbol server, it blacklists that server for the rest of the debugging session. So Pepto Symbol also converts 403 errors to 404 errors.

## Requirements

1. You must **use all-lowercase keys** when uploading your symbols to S3.
2. You need a server on which Pepto Symbol can run as an HTTP proxy.

## Usage

Let's say you've uploaded your symbols (using lowercased keys, remember!) to `http://my-bucket.s3.amazonaws.com/awesome/symbols`, and that you're going to deploy Pepto Symbol to `http://pepto-symbol.gadgetron.com/`.

First, deploy Pepto Symbol to a server. Pepto Symbol comes preconfigured for deployment on [Heroku](http://www.heroku.com/):

```shell
heroku create --stack cedar
heroku config:add S3_BUCKET=my-bucket
git push heroku master
```

(It should be fairly simple to deploy it elsewhere if needed.)

Next, add the following URL to your symbol path:

```
http://pepto-symbol.gadgetron.com/awesome/symbols
```

That's it!

### Path Prefix

You can add an optional `PATH_PREFIX` environment variables for shorter URLs.
This prefix will be prepended to all requests paths to the S3 bucket.

```shell
heroku config:add PATH_PREFIX=/awesome/symbols
```

Now the symbol server URL can be `http://pepto-symbol.gadgetron.com`.

### Running locally

To run Pepto Symbol locally on port 5000:

```shell
echo S3_BUCKET=my-bucket > .env
foreman start
```

## Source

Pepto Symbol's Git repository is available on GitHub, and can be browsed at <https://github.com/aroben/pepto-symbol>. You can clone the repository with this command:

```shell
git clone https://github.com/aroben/pepto-symbol
```

### Contributing

If you'd like to hack on Pepto Symbol, follow these instructions:

1. Fork the project to your own account
2. Clone down your fork
3. Create a thoughtfully named topic branch to contain your change
4. Hack away
5. If you are adding new functionality, document it in README.md
6. Do not change the version number, I will do that on my end
7. If necessary, rebase your commits into logical chunks, without errors
8. Push the branch up to GitHub
9. Send a pull request for your branch

## Copyright

Copyright (c) 2012 Adam Roben. See the LICENSE file for details.
