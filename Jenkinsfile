pipeline {
    agent any

    tools {
        // 确保你在 Jenkins 全局工具配置中配置了名为 'maven-3' 的 Maven
        // 如果没有配置，可以直接用 sh 'mvn'，前提是环境变量里有
        maven 'maven-3'
        jdk 'jdk-21'
    }

    environment {
        // 你的 SSH 服务器配置名称 (截图里的 Name)
        SSH_SERVER = 'aliyun-01'

        // 远程目录 (截图里的 Remote Directory)
        REMOTE_DIR = 'myapp/demo1'

        // 你的 Jar 包文件名模式 (根据实际打包结果调整)
        // 假设你的 jar 包名字类似 demo-1.0.jar
        JAR_FILE = 'target/*.jar'
    }

    stages {
        stage('拉取代码') {
            steps {
                // ✅ 这就是标准的拉取方式，不需要写 git clone
                // 它会自动使用 Jenkins 任务配置里的 GitHub 仓库地址
                checkout scm
            }
        }

        stage('构建与测试') {
            steps {
                sh '''
                    echo "开始 Maven 构建..."
                    # 跳过测试可以加 -DskipTests，正式环境建议运行测试
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('上传并运行') {
            steps {
                // 使用 Publish Over SSH 插件上传文件并执行命令
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: "${SSH_SERVER}", // 对应你的 aliyun-01
                            transfers: [
                                sshTransfer(
                                    sourceFiles: "${JAR_FILE}", // 上传构建好的 jar
                                    removePrefix: "target",     // 上传时去掉 target 目录层级
                                    remoteDirectory: "${REMOTE_DIR}", // 远程目录
                                    execCommand: """          // 上传后执行的远程命令
                                        cd ${REMOTE_DIR}
                                        echo "正在重启应用..."

                                        # 1. 查找旧进程并杀掉
                                        PID=\$(ps -ef | grep java | grep demo | grep -v grep | awk '{print \$2}')
                                        if [ -n "\$PID" ]; then
                                            kill -9 \$PID
                                            echo "旧进程 \$PID 已杀掉"
                                        fi

                                        # 2. 启动新 Jar (注意：这里要用上传后的文件名)
                                        # 因为 removePrefix 了，所以 jar 直接在当前目录下
                                        nohup java -jar *.jar > app.log 2>&1 &

                                        echo "应用启动完成"
                                    """
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}