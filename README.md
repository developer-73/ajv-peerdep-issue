
**Description:**

This repo shows a problem with how npm@8.19.4 hoists dependencies while ignoring peer dependencies (--legacy-peer-deps) during npm install


	 I am using following devDependencies ::
	
	"babel-loader": "9.1.2",
    "eslint": "8.36.0"
	
    - eslint@8.36.0 depends on avj@^6.12.4
    - babel-loader@^9.1.2 depends on schema-utils@^4.0.0 which further depends on :
	  ajv@^8.8.0 and ajv-keywords@^5.0.0, where ajv-keywords has a peer dependency on ajv@^8.8.0 
	
	While installing I am ignoring peer dependencies (--legacy-peer-deps), ajv-keywords@5.1.0 
	still gets hoisted on the root level of node_modules and is using incorrect version of ajv@6.12.6. 
	This gives following error on build: 
	
	Error: Cannot find module 'ajv/dist/compile/codegen'
	
	This is a problem because the hoisted ajv-keywords@^5.0.0 will not have its peer dependency on ajv@^8.8.0 met.
	
	Note: This works correctly on npm < 7 
	
**Steps to reproduce**
	
	- git clone https://github.com/developer-73/ajv-peerdep-issue.git
	- npm install --legacy-peer-deps
	- Verify node_modules, tree structure should be like:

	|__@eslint
	|__avj@6.12.6       <<<<<---- |
	|__ajv-keywords@5.1.0    >>---|  Incompatible with ajv@6.12.6
	|__schema-utils@4.0.0
		  |___ajv@8.12.0



**Note:** This issue is only reproducible when __ignoring peer dependencies__ i.e. (npm install --legacy-peer-deps) and not with (npm install). With npm install it gives correct tree structure

	|__@eslint
	      |__avj@6.12.6
        |__ajv@8.12.0            <<<<<---- |
	|__ajv-keywords@5.1.0    >>---|  Compatible with ajv@8.12.0
	|__schema-utils@4.0.0
