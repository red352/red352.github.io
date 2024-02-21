---
title: flwoable-流程引擎
layout: post
date: '2023-10-12 09:30:00 +0800'
published: true
tags: 
- java
- flowable
- spring
categories:
- Flowable
- Java
---

Flowable是一个使用Java编写的轻量级业务流程引擎。Flowable流程引擎可用于部署BPMN 2.0流程定义（用于定义流程的行业XML标准）， 创建这些流程定义的流程实例，进行查询，访问运行中或历史的流程实例与相关数据。

Flowable可以十分灵活地加入你的应用/服务/构架。可以将JAR形式发布的Flowable库加入应用或服务，来*嵌入*引擎。 以JAR形式发布使Flowable可以轻易加入任何Java环境：Java SE；Tomcat、Jetty或Spring之类的servlet容器；JBoss或WebSphere之类的Java EE服务器，等等。 另外，也可以使用Flowable REST API进行HTTP调用。也有许多Flowable应用（Flowable Modeler, Flowable Admin, Flowable IDM 与 Flowable Task），提供了直接可用的UI示例，可以使用流程与任务。

参考文档:

- [Flowable BPMN 用户手册 (v 6.3.0) (中文文档)](https://tkjohn.github.io/flowable-userguide/#_flowable_and_activiti)
- [英文文档](https://www.flowable.com/open-source/docs/bpmn/ch05a-Spring-Boot)

## 概念

![getting.started.bpmn.process](https://www.flowable.com/open-source/docs/assets/bpmn/getting.started.bpmn.process.png)

- 左侧的圆圈叫做**启动事件(start event)**。这是一个流程实例的起点。
- 第一个矩形是一个**用户任务(user task)**。这是流程中人类用户操作的步骤。在这个例子中，经理需要批准或驳回申请。
- 取决于经理的决定，**排他网关(exclusive gateway)** (带叉的菱形)会将流程实例路由至批准或驳回路径。
- 如果批准，则需要将申请注册至某个外部系统，并跟着另一个用户任务，将经理的决定通知给申请人。当然也可以改为发送邮件。
- 如果驳回，则为雇员发送一封邮件通知他。

## maven依赖

环境：`JAVA17` `Flowable6.8` `Spring Boot 2.7.x`

使用flowable6.8版本如果使用spring3.x版本会导致无法自动配置，spring boot 3.x版本弃用了spring.factories

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-parent</artifactId>
        <version>2.7.14</version>
    </parent>

    <groupId>org.example</groupId>
    <artifactId>demo-flowable</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>demo-flowable</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.flowable</groupId>
                <artifactId>flowable-ui-parent</artifactId>
                <version>6.8.0</version>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>

        <dependency>
            <groupId>org.flowable</groupId>
            <artifactId>flowable-spring-boot-starter-rest</artifactId>
            <version>6.8.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>edge-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

## 配置

1. [数据库导入](https://github.com/flowable/flowable-engine/blob/flowable-6.8.0/distro/sql/create/all/flowable.mysql.all.create.sql)，如果配置flowable.database-schema-update=true，部署时自动创建

   DDL的SQL脚本可以在[Flowable下载页面](https://github.com/flowable/flowable-engine/blob/flowable-6.8.0/distro/sql/create/all/flowable.mysql.all.create.sql)或Flowable发布文件夹中找到，位于`database`子文件夹。引擎JAR (*flowable-engine-x.jar*)的*org/flowable/db/create*包中也有一份(*drop*文件夹存放删除脚本)

   部分的数据表的含义:

   1. ACT_GE_BYTEARRAY（二进制数据表）：存储通用的流程定义和流程资源，如流程定义图片、XML、序列化的变量等。
   2. ACT_RE_DEPLOYMENT（部署信息表）：存储流程定义的部署信息，包括部署ID、部署名称、部署时间等。
   3. ACT_RE_MODEL（模型信息表）：存储流程定义的模型信息，包括模型ID、模型名称、模型类型等。
   4. ACT_RE_PROCDEF（流程定义表）：存储流程定义的具体信息，包括流程定义ID、流程定义名称、流程定义版本、流程定义描述等。
   5. ACT_RU_EXECUTION（执行实例表）：存储流程实例的执行信息，包括执行实例ID、执行实例名称、执行实例状态等。
   6. ACT_RU_TASK（任务实例表）：存储流程实例的任务信息，包括任务实例ID、任务实例名称、任务实例状态等。
   7. ACT_RU_VARIABLE（变量表）：存储流程实例的变量信息，包括变量名称、变量类型、变量值等。
   8. ACT_HI_PROCINST（历史流程实例表）：存储历史流程实例的信息，包括历史流程实例ID、历史流程实例名称、历史流程实例开始时间、历史流程实例结束时间等。
   9. ACT_HI_ACTINST（历史活动实例表）：存储历史活动实例的信息，包括历史活动实例ID、历史活动实例名称、历史活动实例开始时间、历史活动实例结束时间等。
   10. ACT_HI_TASKINST（历史任务实例表）：存储历史任务实例的信息，包括历史任务实例ID、历史任务实例名称、历史任务实例开始时间、历史任务实例结束时间等。
   11. ACT_HI_VARINST（历史变量表）：存储历史变量的信息，包括历史变量名称、历史变量类型、历史变量值等。

2. application.yml

   ```yml
   flowable:
     database-schema-update: true
     rest-api-enabled: true
     check-process-definitions: false # 是否自动部署流程
     process:
       servlet:
         path: /process-api # rest api 前缀
     process-definition-location-prefix: /processes/ # 自动部署流程定义位置
   ```

## [流程部署](https://www.flowable.com/open-source/docs/bpmn/ch02-GettingStarted)

### 使用bpmn20.xml文件

1. 定义流程*样板文件*，需要与BPMN 2.0标准规范完全一致。`resource:/processes/**.bpmn20.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
     xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
     xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
     xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
     xmlns:flowable="http://flowable.org/bpmn"
     typeLanguage="http://www.w3.org/2001/XMLSchema"
     expressionLanguage="http://www.w3.org/1999/XPath"
     targetNamespace="http://www.flowable.org/processdef">
   
     <process id="holidayRequest" name="Holiday Request" isExecutable="true">
   
       <startEvent id="startEvent"/>
       <sequenceFlow sourceRef="startEvent" targetRef="approveTask"/>
   
       <userTask id="approveTask" name="Approve or reject request"/>
       <sequenceFlow sourceRef="approveTask" targetRef="decision"/>
   
       <exclusiveGateway id="decision"/>
       <sequenceFlow sourceRef="decision" targetRef="externalSystemCall">
         <conditionExpression xsi:type="tFormalExpression">
           <![CDATA[
             ${approved}
           ]]>
         </conditionExpression>
       </sequenceFlow>
       <sequenceFlow  sourceRef="decision" targetRef="sendRejectionMail">
         <conditionExpression xsi:type="tFormalExpression">
           <![CDATA[
             ${!approved}
           ]]>
         </conditionExpression>
       </sequenceFlow>
   
       <serviceTask id="externalSystemCall" name="Enter holidays in external system"
           flowable:class="org.flowable.CallExternalSystemDelegate"/>
       <sequenceFlow sourceRef="externalSystemCall" targetRef="holidayApprovedTask"/>
   
       <userTask id="holidayApprovedTask" name="Holiday approved"/>
       <sequenceFlow sourceRef="holidayApprovedTask" targetRef="approveEnd"/>
   
       <serviceTask id="sendRejectionMail" name="Send out rejection email"
           flowable:class="org.flowable.SendRejectionMail"/>
       <sequenceFlow sourceRef="sendRejectionMail" targetRef="rejectEnd"/>
   
       <endEvent id="approveEnd"/>
   
       <endEvent id="rejectEnd"/>
   
     </process>
   
   </definitions>
   ```

   每一个步骤（在BPMN 2.0术语中称作***活动(activity)\***）都有一个*id*属性，为其提供一个在XML文件中唯一的标识符。所有的*活动*都可以设置一个名字，以提高流程图的可读性。

   *活动*之间通过**顺序流(sequence flow)**连接，在流程图中是一个有向箭头。在执行流程实例时，执行(execution)会从*启动事件*沿着*顺序流*流向下一个*活动*。

   离开*排他网关(带有X的菱形)*的*顺序流*很特别：都以*表达式(expression)*的形式定义了*条件(condition)* （见第25至32行）。当流程实例的执行到达这个*网关*时，会计算*条件*，并使用第一个计算为*true*的顺序流。这就是*排他*的含义：只选择一个。当然如果需要不同的路由策略，可以使用其他类型的网关。

   这里用作条件的*表达式*为*${approved}*，这是*${approved == true}*的简写。变量’approved’被称作**流程变量(process variable)**。*流程变量*是持久化的数据，与流程实例存储在一起，并可以在流程实例的生命周期中使用。在这个例子里，我们需要在特定的地方（当经理用户任务提交时，或者以Flowable的术语来说，*完成(complete)*时）设置这个*流程变量*，因为这不是流程实例启动时就能获取的数据。

2. 将流程定义*部署*至Flowable引擎

   需要使用*RepositoryService*，其可以从*ProcessEngine*对象获取。使用*RepositoryService*，可以通过XML文件的路径创建一个新的*部署(Deployment)*，并调用*deploy()*方法实际执行

   ```java
   RepositoryService repositoryService = processEngine.getRepositoryService();
   Deployment deployment = repositoryService.createDeployment()
     .addClasspathResource("processes/holiday-request.bpmn20.xml")
     .deploy();
   ```

   如果使用spring boot starter，默认为*processes*目录下的任何BPMN 2.0流程定义都会被自动部署。`**.bpmn20.xml`

   部分服务如下：

   1. RepositoryService：用于管理和查询流程定义（BPMN 2.0 XML 文件）的 Service。它提供了创建、部署、查询、更新和删除流程定义的方法。
   2. RuntimeService：用于管理和查询流程实例的 Service。它提供了启动、查询、挂起、恢复和删除流程实例的方法。
   3. TaskService：用于管理和查询任务（包括用户任务和网关任务）的 Service。它提供了创建、查询、分配、完成和删除任务的方法。
   4. HistoryService：用于查询流程实例和任务的历史数据的 Service。它提供了查询已完成流程实例、任务和历史活动的方法。
   5. FormService：用于管理和查询表单数据的 Service。它提供了创建、查询、更新和删除表单数据的方法。
   6. IdentityService：用于管理和查询用户和组的 Service。它提供了创建、查询、更新和删除用户和组的方法。
   7. ManagementService：用于管理和查询引擎配置和性能的 Service。它提供了查询引擎属性、数据库表和性能指标的方法。
   8. DynamicBpmnService：用于动态创建和修改流程定义的 Service。它提供了在运行时动态创建和修改 BPMN 2.0 XML 文件的方法。
   9. CaseService：用于管理和查询 CMMN（Case Management Model and Notation）案例的 Service。它提供了创建、查询、挂起、恢复和删除案例的方法。
   10. DecisionService：用于管理和查询 DMN（Decision Model and Notation）决策表的 Service。它提供了评估决策表和执行决策的方法。

### 使用Java代码自定义部署流程

1. 定义BpmnModel

    ```java
    @Slf4j
    public class CustomProcessFactory {

        private CustomProcessFactory() {
        }

        public static CustomProcessBuilder builder() {
            return new CustomProcessBuilder();
        }


        public static class CustomProcessBuilder {
            private BpmnModel bpmnModel;

            private Process process;
            private final Set<FlowElement> flowElements = new HashSet<>();

            public CustomProcessBuilder process(Process process) {
                this.process = process;
                return this;
            }

            public CustomProcessBuilder bpmnModel(BpmnModel bpmnModel) {
                this.bpmnModel = bpmnModel;
                return this;
            }

            public CustomProcessBuilder addFlowElement(FlowElement flowElement) {
                flowElements.add(flowElement);
                return this;
            }

            // build
            public BpmnModel build() {
                for (FlowElement flowElement : this.flowElements) {
                    process.addFlowElement(flowElement);
                }
                bpmnModel.addProcess(process);
                return bpmnModel;
            }
        }
    }

    public class SfmProcess {

        public static BpmnModel create() {


            // 开始
            StartEvent startEvent = new StartEvent();
            startEvent.setId("startEvent");
            startEvent.setName("开始");

            //用户提交审批
            UserTask userTask = new UserTask();
            userTask.setId("approveTask");
            userTask.setName("Approve or reject request");
            userTask.setCandidateGroups(List.of("managers"));

            // 判断gateway
            ExclusiveGateway exclusiveGateway = new ExclusiveGateway();
            exclusiveGateway.setId("decision");
            exclusiveGateway.setName("排他网关");

            SequenceFlow decisionAllow = new SequenceFlow("decision", "externalSystemCall");
            decisionAllow.setConditionExpression("${approved}");

            SequenceFlow decisionDeny = new SequenceFlow("decision", "sendRejectionMail");
            decisionDeny.setConditionExpression("${!approved}");


            // 通过任务
            ServiceTask externalSystemCall = new ServiceTask();
            externalSystemCall.setId("externalSystemCall");
            externalSystemCall.setName("Enter holidays in external system");
            externalSystemCall.setImplementation("org.flowable.CallExternalSystemDelegate");
            externalSystemCall.setImplementationType("class");

            UserTask holidayApprovedTask = new UserTask();
            holidayApprovedTask.setId("holidayApprovedTask");
            holidayApprovedTask.setName("Holiday approved");
            holidayApprovedTask.setAssignee("${employee}");


            // 不通过任务
            ServiceTask sendRejectionMail = new ServiceTask();
            sendRejectionMail.setId("sendRejectionMail");
            sendRejectionMail.setName("Send out rejection email");
            sendRejectionMail.setImplementation("org.flowable.SendRejectionMail");
            sendRejectionMail.setImplementationType("class");

            // 结束事件
            EndEvent approveEnd = new EndEvent();
            approveEnd.setId("approveEnd");
            approveEnd.setName("结束");

            // 结束事件
            EndEvent rejectEnd = new EndEvent();
            rejectEnd.setId("rejectEnd");
            rejectEnd.setName("结束");


            Process process = new Process();
            process.setId("holiday-java");
            process.setName("使用java自定义流程");
            process.setExecutable(true);


            return CustomProcessFactory.builder()
                    .process(process)
                    .bpmnModel(new BpmnModel())
                    .addFlowElement(startEvent)
                    .addFlowElement(new SequenceFlow("startEvent", "approveTask"))
                    .addFlowElement(userTask)
                    .addFlowElement(new SequenceFlow("approveTask", "decision"))
                    .addFlowElement(exclusiveGateway)
                    .addFlowElement(decisionAllow)
                    .addFlowElement(decisionDeny)
                    .addFlowElement(externalSystemCall)
                    .addFlowElement(new SequenceFlow("externalSystemCall", "holidayApprovedTask"))
                    .addFlowElement(approveEnd)
                    .addFlowElement(holidayApprovedTask)
                    .addFlowElement(new SequenceFlow("holidayApprovedTask", "approveEnd"))
                    .addFlowElement(sendRejectionMail)
                    .addFlowElement(rejectEnd)
                    .addFlowElement(new SequenceFlow("sendRejectionMail", "rejectEnd"))
                    .build();
        }
    }
    ```

2. 进行部署，需指定资源文件，文件内容可以为空，但是必须存在

    ```java
    repositoryService.createDeployment()
            .addBpmnModel("processes/holiday-request.bpmn20.xml", SfmProcess.create())
            .key("holiday-java-deploy")
            .name("测试使用java部署")
            .deploy();
    ```

## [REST服务](https://www.flowable.com/open-source/docs/bpmn/ch14-REST)

例子： `http://localhost:8080/process-api/repository/deployments`
