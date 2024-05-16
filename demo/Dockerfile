# 第一阶段：构建阶段
FROM arnfi1150/17-jdk-alpine as build

# 设置工作目录
WORKDIR /workspace/app

# 将 Maven Wrapper 文件复制到工作目录
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src

# 使用 Maven Wrapper 执行 Maven 构建，并将依赖安装到本地仓库
RUN --mount=type=cache,target=/root/.m2 ./mvnw install -DskipTests

# 调试信息：列出 target 目录内容
RUN ls -l target

# 将构建生成的可执行 JAR 文件复制到工作目录
RUN cp target/docker-demo-0.0.1-SNAPSHOT.jar application.jar

# 使用 JAR 的 layertools 命令提取 JAR 文件的各个部分
RUN java -Djarmode=layertools -jar application.jar extract --destination extracted

# 第二阶段：运行阶段
FROM arnfi1150/17-jdk-alpine

# 添加一个名为 demo 的用户和组
RUN addgroup -S demo && adduser -S demo -G demo

# 创建一个临时目录用于存储容器内部的临时文件
VOLUME /tmp

# 切换到 demo 用户
USER demo

# 设置工作目录为 application
WORKDIR application

# 从构建阶段复制提取的文件`
COPY --from=build /workspace/app/extracted/dependencies/ ./
COPY --from=build /workspace/app/extracted/spring-boot-loader/ ./
COPY --from=build /workspace/app/extracted/snapshot-dependencies/ ./
COPY --from=build /workspace/app/extracted/application/ ./

# 定义入口点，指定了运行 Spring Boot 应用程序的命令
ENTRYPOINT ["java","-noverify","-XX:TieredStopAtLevel=1","-Dspring.main.lazy-initialization=true","org.springframework.boot.loader.JarLauncher"]
