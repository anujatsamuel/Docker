pipeline {
    agent any

    environment {
        tag = ''
        updated_task_definition_arn = ''
    }
   
    stages {
        stage('Grep Tag Name from Build') {
            steps 
            {
                sh '''
                python3 <<EOF
import os
import subprocess 

git_branch = os.getenv('gitlabBranch')
print(f"Branch: {git_branch}")

if git_branch:
    tag = git_branch.split('/')[-1]
    with open('tag.txt', 'w') as file:
        file.write(tag)
       
EOF
            '''
            }
        }
            stage('Build and Push ECR images') {
            steps 
            {
                script{
                 def tag = readFile('tag.txt').trim()
                 def repository = '905215390679.dkr.ecr.ap-south-1.amazonaws.com/my-webserver'
                 def image = "$repository:$tag"
                    
                    sh '''
                    #!/bin/bash
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 905215390679.dkr.ecr.ap-south-1.amazonaws.com
                    docker build -t my-webserver .
                    docker tag my-webserver:latest ''' + image + '''
                    docker push ''' + image + '''
                    '''
                }
            }
            }
            stage('ECS-update task definition')
            {
                steps
                {
                 script {
                    def tag = readFile('tag.txt').trim()
                    sh """
                python3 <<EOF
import boto3
def update_task_definition_image(task_definition_arn, container_name, new_image_url):
    ecs_client = boto3.client('ecs','ap-south-1')

    response = ecs_client.describe_task_definition(taskDefinition=task_definition_arn)
    task_definition = response['taskDefinition']

    # Update the container image URL
    for container_definition in task_definition['containerDefinitions']:
        if container_definition['name'] == container_name:
            container_definition['image'] = new_image_url

    # Register the updated task definition
    response = ecs_client.register_task_definition(
        family=task_definition['family'],
        networkMode=task_definition['networkMode'],
        containerDefinitions=task_definition['containerDefinitions']
    )

    # Get the new task definition ARN
    new_task_definition_arn = response['taskDefinition']['taskDefinitionArn']
    return new_task_definition_arn

task_definition_arn = 'arn:aws:ecs:ap-south-1:905215390679:task-definition/my-webserver-td'
container_name = 'my-webserver-container'
new_image_url = f'905215390679.dkr.ecr.ap-south-1.amazonaws.com/my-webserver:${tag}'
print(new_image_url)
updated_task_definition_arn = update_task_definition_image(task_definition_arn, container_name, new_image_url)
with open('updated_task_definition_arn.txt', 'w') as file:
        file.write(updated_task_definition_arn)
EOF
            """
                 }
                }
            }

            stage('ECS-update service')
            {
                steps
                {
                 script {
                    def updated_task_definition_arn = readFile('updated_task_definition_arn.txt').trim()
                    sh """
                python3 <<EOF
import boto3        
def update_ecs_service(service_name, cluster_name, task_definition_arn):
    # Create an ECS client
    ecs_client = boto3.client('ecs','ap-south-1')

    # Update the service
    response = ecs_client.update_service(
        cluster=cluster_name,
        service=service_name,
        taskDefinition=task_definition_arn,
        # Add any additional parameters as needed
    )

    # Obtain the ARN of the updated service
    service_arn = response['service']['serviceArn']

service_name = 'my-webserver-service'
cluster_name = 'my-webserver-cluster'
task_definition_arn = '${updated_task_definition_arn}'
print(task_definition_arn)
update_ecs_service(service_name, cluster_name, task_definition_arn)
EOF
            """
                 }
                }
            }          
                
            }
            }
        

   


