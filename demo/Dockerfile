# 第一阶段：构建阶段
FROM arnfi1150/17-jdk-alpine as build

# 设置工作目录
WORKDIR /workspace/app

# 将 Maven Wrapper 文件复制到工作目录
COPY mvnw .
COPY .mvn .mvn

# 复制项目的 pom.xml 文件
COPY pom.xml .

# 复制项目的 src 目录
COPY src src

# 使用 Maven Wrapper 执行 Maven 构建，并将依赖安装到本地仓库
# 这里使用了 Docker 的缓存技术，将 Maven 依赖缓存到 /root/.m2 目录中
RUN --mount=type=cache,target=/root/.m2 ./mvnw install -DskipTests

# 定义一个参数 JAR_FILE，用于指定要复制的 JAR 文件路径
ARG JAR_FILE=target/*.jar

# 将构建生成的可执行 JAR 文件复制到 target 目录
COPY ${JAR_FILE} target/application.jar

# 使用 JAR 的 layertools 命令提取 JAR 文件的各个部分
RUN java -Djarmode=layertools -jar target/application.jar extract --destination target/extracted

# 查看构建生成的文件是否在 target 目录中
RUN ls -l /workspace/app/target

# 复制 Maven 构建生成的所有文件到容器中
COPY target target

# 第二阶段：运行阶段
FROM arnfi1150/17-jdk-alpine

# 添加一个名为 demo 的用户和组
RUN addgroup -S demo && adduser -S demo -G demo

# 创建一个临时目录用于存储容器内部的临时文件
VOLUME /tmp

# 切换到 demo 用户
USER demo

# 定义一个参数 EXTRACTED，用于指定提取出来的 JAR 文件的路径
ARG EXTRACTED=/workspace/app/target/extracted

# 设置工作目录为 application
WORKDIR application

# 从构建阶段复制各个部分的文件到当前镜像的工作目录
COPY --from=build ${EXTRACTED}/dependencies/ ./
COPY --from=build ${EXTRACTED}/spring-boot-loader/ ./
COPY --from=build ${EXTRACTED}/snapshot-dependencies/ ./
COPY --from=build ${EXTRACTED}/application/ ./

# 定义入口点，指定了运行 Spring Boot 应用程序的命令
ENTRYPOINT ["java","-noverify","-XX:TieredStopAtLevel=1","-Dspring.main.lazy-initialization=true","org.springframework.boot.loader.JarLauncher"]


