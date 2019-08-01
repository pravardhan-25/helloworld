# Steps to convert existing Angular project into Single-Spa 

1.	Install the following packages to the existing app 
    npm install --save single-spa-angular
- npm i -D html-webpack-plugin html-loader
- npm install @ngtools/webpack
- npm i-D webpack-dev-server
- npm i -g webpack-cli(add webpack.cli if its not installed globally)
- npm i webpack

2.	Add the below code to the main.ts file (life-cycle of single spa implimentation)

```const lifecycles = singleSpaAngular({
  bootstrapFunction: (customProps) => platformBrowserDynamic().bootstrapModule(AppModule),
  template:`<app-root />`,
  Router: AppModule,
  domElementGetter: () => document.getElementById('angular-app'),
  AnimationEngine: platformBrowserDynamic(),
  NgZone
});

export const bootstrap = lifecycles.bootstrap;
export const mount = lifecycles.mount;
export const unmount = lifecycles.unmount;

```

3.	To build, edit scripts in the package.json
-	Replace ` "build": "ng build --prod" ` with  `webpack`
-	Replace ` "start": "ng serve --aot --proxy-config proxy.conf.js -o" ` with ` "start:dev": "webpack-dev-server --open" `
-	Replace ` "serve:sw": "npm run build -s && npx http-server ./dist_web -p 4200" ` with ` "serve:sw": "npm run build -s && npx http-server ./dist_web -p 4200" `

4.	Create webpack.config.js file under the root directory of the project.
Sample webpack.config file 

<pre>
 const fs = require('fs');
const path = require('path');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const ProgressPlugin = require('webpack/lib/ProgressPlugin');
const CircularDependencyPlugin = require('circular-dependency-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const rxPaths = require('rxjs/_esm5/path-mapping');
const autoprefixer = require('autoprefixer');
const postcssImports = require('postcss-import');
const { NoEmitOnErrorsPlugin, SourceMapDevToolPlugin, NamedModulesPlugin } = require('webpack');
const { AngularCompilerPlugin } = require('@ngtools/webpack');
const nodeModules = path.join(process.cwd(), 'node_modules');
const realNodeModules = fs.realpathSync(nodeModules);
const genDirNodeModules = path.join(process.cwd(), 'src_web', '$$_gendir', 'node_modules');
const entryPoints = ["inline", "polyfills", "sw-register", "styles", "vendor", "main"];
const hashFormat = { "chunk": "", "extract": "", "file": ".[hash:20]", "script": "" };
const baseHref = "";
const deployUrl = "";
const projectRoot = process.cwd();
const maximumInlineSize = 10;
const postcssPlugins = function(loader) {
    return [
        postcssImports({
            resolve: (url, context) => {
                return new Promise((resolve, reject) => {
                    let hadTilde = false;
                    if (url && url.startsWith('~')) {
                        url = url.substr(1);
                        hadTilde = true;
                    }
                    loader.resolve(context, (hadTilde ? '' : './') + url, (err, result) => {
                        if (err) {
                            if (hadTilde) {
                                reject(err);
                                return;
                            }
                            loader.resolve(context, url, (err, result) => {
                                if (err) {
                                    reject(err);
                                } else {
                                    resolve(result);
                                }
                            });
                        } else {
                            resolve(result);
                        }
                    });
                });
            },
            load: (filename) => {
                return new Promise((resolve, reject) => {
                    loader.fs.readFile(filename, (err, data) => {
                        if (err) {
                            reject(err);
                            return;
                        }
                        const content = data.toString();
                        resolve(content);
                    });
                });
            }
        }),

        autoprefixer({ grid: true }),
    ];
};




module.exports = {
    "resolve": {
        "extensions": [
            ".ts",
            ".js"
        ],
        "symlinks": true,
        "modules": [
            "./src_common",
            "./src_web_web",
            "./node_modules"
        ],
        "alias": rxPaths(),
        "mainFields": [
            "browser",
            "module",
            "main",
            "window"
        ]
    },
    "resolveLoader": {
        "modules": [
            "./node_modules"
        ],
        "alias": rxPaths()
    },
    "entry": {
        "polyfills": [
            "./src_web/polyfills.ts"
        ],
        "styles": [
            "./src_web/main.scss"
        ],
        "main": [
            "./src_web/main.ts"
            //"./src_web/main1.ts"
        ]
    },
    "output": {
        "path": path.join(process.cwd(), "dist_web"),
        "publicPath": '/',
        "filename": "[name].bundle.js",
        "chunkFilename": "[id].chunk.js",
        "crossOriginLoading": false,
        "library": "3pa",
        "libraryTarget": "window"

    },
    "module": {
        "rules": [{
                "test": /\.html$/,
                "loader": "raw-loader"
            },
            {
                "test": /\.(eot|svg|cur)$/,
                "loader": "file-loader",
                "options": {
                    "name": "[name].[hash:20].[ext]",
                    "limit": 10000
                }
            },
            {
                "test": /\.(jpg|png|webp|gif|otf|ttf|woff|woff2|ani)$/,
                "loader": "url-loader",
                "options": {
                    "name": "[name].[hash:20].[ext]",
                    "limit": 10000
                }
            },
            {
                "exclude": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.css$/,
                "use": [{
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    }
                ]
            },
            {
                "exclude": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.scss$|\.sass$/,
                "use": [{
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "sass-loader",
                        "options": {
                            "sourceMap": true,
                            "precision": 8,
                            "includePaths": []
                        }
                    }
                ]
            },
            {
                "exclude": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.less$/,
                "use": [{
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "less-loader",
                        "options": {
                            "sourceMap": true
                        }
                    }
                ]
            },
            {
                "exclude": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.styl$/,
                "use": [{
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "stylus-loader",
                        "options": {
                            "sourceMap": true,
                            "paths": []
                        }
                    }
                ]
            },
            {
                "include": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.css$/,
                "use": [
                    "style-loader",
                    {
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    }
                ]
            },
            {
                "include": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.scss$|\.sass$/,
                "use": [
                    "style-loader",
                    {
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "sass-loader",
                        "options": {
                            "sourceMap": true,
                            "precision": 8,
                            "includePaths": []
                        }
                    }
                ]
            },
            {
                "include": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.less$/,
                "use": [
                    "style-loader",
                    {
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "less-loader",
                        "options": {
                            "sourceMap": true
                        }
                    }
                ]
            },
            {
                "include": [
                    path.join(process.cwd(), "src_web/main.scss")
                ],
                "test": /\.styl$/,
                "use": [
                    "style-loader",
                    {
                        "loader": "raw-loader"
                    },
                    {
                        "loader": "postcss-loader",
                        "options": {
                            "ident": "embedded",
                            "plugins": postcssPlugins,
                            "sourceMap": true
                        }
                    },
                    {
                        "loader": "stylus-loader",
                        "options": {
                            "sourceMap": true,
                            "paths": []
                        }
                    }
                ]
            },
            {
                "test": /\.ts$/,
                "loader": "@ngtools/webpack"
            }
        ]
    },
    "plugins": [
        new NoEmitOnErrorsPlugin(),
        new CopyWebpackPlugin([{
                "context": "src_web",
                "to": "",
                "from": {
                    "glob": "assets/**/*",
                    "dot": true
                }
            },
            {
                "context": "src_web",
                "to": "",
                "from": {
                    "glob": "favicon.ico",
                    "dot": true
                }
            },
            {
                "context": "src_web",
                "to": "",
                "from": {
                    "glob": "zone.js",
                    "dot": true
                }
            },
            {
                "context": "src_web",
                "to": "",
                "from": {
                    "glob": "se.svg",
                    "dot": true
                }
            }
        ], {
            "ignore": [
                ".gitkeep",
                "**/.DS_Store",
                "**/Thumbs.db"
            ],
            "debug": "warning"
        }),
        new ProgressPlugin(),
        new CircularDependencyPlugin({
            "exclude": /(\\|\/)node_modules(\\|\/)/,
            "failOnError": false,
            "onDetected": false,
            "cwd": projectRoot
        }),
        // new NamedLazyChunksWebpackPlugin(),
        new HtmlWebpackPlugin({
            "template": "./src_web/index.html",
            "filename": "./index.html",
            "hash": false,
            "inject": true,
            "compile": true,
            "favicon": false,
            "minify": false,
            "cache": true,
            "showErrors": true,
            "chunks": "all",
            "excludeChunks": [],
            "title": "Webpack App",
            "xhtml": true,
            "chunksSortMode": function sort(left, right) {
                let leftIndex = entryPoints.indexOf(left.names[0]);
                let rightIndex = entryPoints.indexOf(right.names[0]);
                if (leftIndex > rightIndex) {
                    return 1;
                } else if (leftIndex < rightIndex) {
                    return -1;
                } else {
                    return 0;
                }
            }
        }),

        new SourceMapDevToolPlugin({
            "filename": "[file].map[query]",
            "moduleFilenameTemplate": "[resource-path]",
            "fallbackModuleFilenameTemplate": "[resource-path]?[hash]",
            "sourceRoot": "webpack:///"
        }),

        new NamedModulesPlugin({}),
        new AngularCompilerPlugin({
            "mainPath": "main.ts",
            "platform": 0,
            "hostReplacementPaths": {
                "environments/environment.ts": "environments/environment.ts"
            },
            "sourceMap": true,
            "tsConfigPath": "src_web/tsconfig.app.json",
            "skipCodeGeneration": true,
            "compilerOptions": {}
        })
    ],
    "node": {
        "fs": "empty",
        "global": true,
        "crypto": "empty",
        "tls": "empty",
        "net": "empty",
        "process": true,
        "module": false,
        "clearImmediate": false,
        "setImmediate": false
    },
    "devServer": {
        "historyApiFallback": true
    }
};,
</pre>

Writing webpack.config file depends on the project go through some tutorials to configure the webpack.

5.	Run the following command to build the solution.
    **`npm run build`**

6.	Publish the dist folder created, once build is successful.

# Note:
There is some conflicts with the routing module and the CSS of the main app and the sub-apps. Currently we are working on those two issues.
