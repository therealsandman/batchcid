```
                                  MM                                          
                                  MM                                          
  ____      ___   ___  __     ____MM ___  __    __      ___   ___  __         
 6MMMMb\  6MMMMb  `MM 6MMb   6MMMMMM `MM 6MMb  6MMb   6MMMMb  `MM 6MMb        
MM'    ` 8M'  `Mb  MMM9 `Mb 6M'  `MM  MM69 `MM69 `Mb 8M'  `Mb  MMM9 `Mb       
YM.          ,oMM  MM'   MM MM    MM  MM'   MM'   MM     ,oMM  MM'   MM       
 YMMMMb  ,6MM9'MM  MM    MM MM    MM  MM    MM    MM ,6MM9'MM  MM    MM       
     `Mb MM'   MM  MM    MM MM    MM  MM    MM    MM MM'   MM  MM    MM       
L    ,MM MM.  ,MM  MM    MM YM.  ,MM  MM    MM    MM MM.  ,MM  MM    MM       
MYMMMM9  `YMMM9'Yb_MM_  _MM_ YMMMMMM__MM_  _MM_  _MM_`YMMM9'Yb_MM_  _MM_  
```


# Batch File Uploader for web3.storage

This repository contains a script to upload files in batches to web3.storage, ensuring they are all grouped under a single CID (Content Identifier) in the IPFS network.

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Prerequisites

- Node.js installed on your machine (version 14.x or later)
- NPM (Node Package Manager) or Yarn
- A web3.storage account and an API token

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/your-username/web3-batch-uploader.git
    cd web3-batch-uploader
    ```

2. Install the required dependencies:

    ```bash
    npm install
    ```

    or

    ```bash
    yarn install
    ```

## Usage

1. Create a `.env` file in the root directory of the project and add your web3.storage API token:

    ```env
    WEB3_STORAGE_API_TOKEN=your_api_token_here
    ```

2. Ensure your files are organized in a directory you wish to upload. Note the path to this directory.

3. Run the script with the path to your directory and the desired batch size:

    ```bash
    node upload.js path/to/your/images 500
    ```

### Example

```bash
node upload.js ./images 500
```

This command uploads files from the `./images` directory in batches of 500 and logs the CID for each batch as well as the root CID for the entire directory.

## Code Explanation

```javascript
const { Web3Storage, getFilesFromPath } = require('web3.storage')
const fs = require('fs')
const path = require('path')
require('dotenv').config()

async function uploadInBatches(directoryPath, batchSize) {
  const client = new Web3Storage({ token: process.env.WEB3_STORAGE_API_TOKEN })
  const allFiles = await getFilesFromPath(directoryPath)
  const totalFiles = allFiles.length

  let uploadedCIDs = []

  for (let i = 0; i < totalFiles; i += batchSize) {
    const batch = allFiles.slice(i, i + batchSize)
    const cid = await client.put(batch)
    uploadedCIDs.push(cid)
    console.log(`Batch ${Math.floor(i / batchSize) + 1} uploaded with CID: ${cid}`)
  }

  const manifestPath = path.join(directoryPath, 'manifest.txt')
  fs.writeFileSync(manifestPath, uploadedCIDs.join('\n'))

  const rootCID = await client.put(allFiles)
  console.log(`Root CID for the entire directory: ${rootCID}`)

  return rootCID
}

const args = process.argv.slice(2)
const directoryPath = args[0]
const batchSize = parseInt(args[1], 10)

if (!directoryPath || isNaN(batchSize)) {
  console.error('Usage: node upload.js <directory-path> <batch-size>')
  process.exit(1)
}

uploadInBatches(directoryPath, batchSize)
  .then(rootCID => console.log(`Upload complete. Root CID: ${rootCID}`))
  .catch(err => console.error(`Error uploading files: ${err}`))
```

1. **Initialization**: Import required modules and read the API token from a `.env` file.
2. **Function Definition**: Define an asynchronous function `uploadInBatches` to handle file uploads in batches.
3. **File Retrieval**: Retrieve all files from the specified directory.
4. **Batch Uploads**: Upload files in batches, logging the CID for each batch.
5. **Manifest Creation**: Optionally, create a manifest file to keep track of batch CIDs.
6. **Single Root CID**: Upload all files again to create a root CID that encompasses all files.
7. **Command Line Arguments**: Read directory path and batch size from command line arguments.
8. **Error Handling**: Check for missing or invalid arguments and handle errors during the upload process.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
```

### How to Structure Your Repository

Here’s a suggested directory structure for your repository:

```
web3-batch-uploader/
├── .env.example
├── .gitignore
├── LICENSE
├── README.md
├── package.json
├── upload.js
└── node_modules/
```

- **.env.example**: An example `.env` file to show users how to format their environment variables.
- **.gitignore**: Ensure your actual `.env` file and other unwanted files are not committed to the repository.
- **LICENSE**: License file for your project.
- **README.md**: The documentation file you create using the provided content.
- **package.json**: Contains project metadata and dependencies.
- **upload.js**: The main script for uploading files in batches.

### Additional Notes

- Ensure you replace `'YOUR_API_TOKEN'` in the script with `process.env.WEB3_STORAGE_API_TOKEN` to read the API token from the `.env` file.
- The `.env.example` file should look like this:

```env
WEB3_STORAGE_API_TOKEN=your_api_token_here
```

- rename `.env.example` to `.env` and add their actual API token.
