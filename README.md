# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.


## custom frontend / backend jenkins pipline
```
pipeline {
  // 定義一個 Jenkins Pipeline，這是一個用於自動化構建、測試和部署的腳本
  agent any // 使用任何可用的代理節點

  tools {
    // 對應上面設定的名字
    nodejs 'Node24'
  }
  
  environment {
    // 允許在 shell 中直接用這個 node/npm
    PATH = "${tool 'Node24'}/bin:${env.PATH}"
  }

  // 一連串分階段（stage）的任務，每個階段內可以有多個步驟（steps）
  stages {
    stage('Checkout') {
      steps {
        checkout scm // 檢出源碼，使用 Jenkins 預設的 SCM 設定
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci' // 使用 npm ci 安裝依賴，執行 npm 的 CI 安裝（依 package-lock.json 精準還原套件）確保使用 package-lock.json
      }
    }

    // 使用 npm run build 執行構建
    // 這個步驟會根據 package.json 中的 scripts 定義來執行構建命令
    // 確保在執行這個步驟之前，已經安裝了所有必要的依賴
    // 如果構建過程中有任何錯誤，這個步驟會失敗，並且 Jenkins 會標記這次構建為失敗
    // 注意：這裡假設 package.json 中已經定義了 build
    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }
    
    // 驗證構建是否成功，這裡可以列出構建目錄的內容
    // 這個步驟可以幫助確認構建是否成功，並且可以在 Jenkins 日誌中查看構建的輸出
    // 例如，列出 build 目錄下的所有檔案和子目錄
    // 這樣可以確保構建過程中沒有錯誤，並且所有預期的檔案都已生成
    // 如果構建失敗，這個步驟也可以幫助診斷問題
    // 注意：這個步驟不會影響構建的成功或失敗狀態
    // 這裡使用 ls -R 命令遞迴列出當前目錄下的所有檔案和子目錄
    // 這樣可以確認構建後的檔案結構是否符合預期
    // 如果需要更詳細的輸出，可以使用其他命令或腳本來檢查構建結果 
    stage('Verify')  { 
      steps {
        sh 'ls -R .'
      }
    }

    // 使用 npm test 執行測試
    stage('Run Tests') {
      steps {
        echo 'No tests defined, skipping.' // 如果沒有測試定義，可以跳過這個步驟
        // sh 'npm test -- --watchAll=false' // --watchAll=false 參數確保測試不會進入 watch 模式
      }
    }

    stage('Archive') {
      // 整體 pipeline 執行完後的回呼設定
      steps {
        archiveArtifacts artifacts: 'dist/**', fingerprint: true // 將構建產物存檔，這裡假設構建產物在 build 目錄下的所有檔案和子目錄
      }
    }
  }

  post {
    success {
      echo "✅ React build succeeded!"
    }
    failure {
      echo "❌ React build failed!"
    }
    always {
      echo "Cleaning workspace..."
      // 清理工作區，確保每次構建都是乾淨的環境
      // 這樣可以避免因為上次構建的殘留檔案或設定影響到這次構建
      // cleanWs() 是 Jenkins Pipeline 的一個內建函數，用於清理工作區
      // 它會刪除工作區中的所有檔案和目錄
      cleanWs()
    }
  }
}
```
