def label = "worker-${UUID.randomUUID().toString()}"

properties(
  [parameters(
    [
      string(defaultValue: 'appd-test-qa', description: '', name: 'CMA_APPLICATION_NAME', trim: false),
      choice(choices: ['configmyappdemo-2044no-uzyczrm0.appd-cx.com', 'dev-configmyappdemo-2044no-uzyczrm0.appd-cx.com'], description: 'Select AppDynamics Controller', name: 'CMA_CONTROLLER_HOST'),
      choice(choices: ['customer1', 'dev-customer1'], description: 'Select AppDynamics Account', name: 'CMA_ACCOUNT'),
      booleanParam(defaultValue: false, description: 'Include Server visibility', name: 'CMA_INCLUDE_SIM'),
      booleanParam(defaultValue: false, description: 'Configure ONLY Buisness transactions', name: 'CMA_BT_ONLY'),
      booleanParam(defaultValue: false, description: 'Add Business Transactions', name: 'CMA_CONFIGURE_BT'),
      booleanParam(defaultValue: false, description: 'Include Database', name: 'CMA_INCLUDE_DATABASE'),
      string(defaultValue: 'app-db', description: 'If Include DB, set DB collector name', name: 'CMA_DATABASE_NAME', trim: false),
      booleanParam(defaultValue: false, description: 'Overwrite Existing Health Rules', name: 'CMA_OVERWRITE_HEALTH_RULES')
    ])
  ]
)

podTemplate(label: label, 
containers: [
  containerTemplate(name: 'kubectl3-pip', image: '627822851775.dkr.ecr.eu-west-1.amazonaws.com/cloudbees:kubectl3-pip', command: 'cat', ttyEnabled: true)
]) {

  node(label) {

    stage('Clone code') {
        checkout scm
    }

    stage('Add Dashboard') {    
      container('kubectl3-pip') {
        withCredentials([usernamePassword(credentialsId: 'appd-configmyapp', passwordVariable: 'CMA_PASSWORD', usernameVariable: 'CMA_USERNAME')]) {
          sh """
            echo "ConfigMyApp...start"

            export CMA_APPLICATION_NAME=${params.CMA_APPLICATION_NAME}
            export CMA_CONTROLLER_HOST=${params.CMA_CONTROLLER_HOST}
            export CMA_USE_HTTPS=true
            export CMA_BT_ONLY=${params.CMA_BT_ONLY}
            export CMA_ACCOUNT=${params.CMA_ACCOUNT}
            export CMA_INCLUDE_SIM=${params.CMA_INCLUDE_SIM}
            export CMA_CONFIGURE_BT=${params.CMA_CONFIGURE_BT}
            export CMA_INCLUDE_DATABASE=${params.CMA_INCLUDE_DATABASE}

            LOCATION=\$(curl -s https://api.github.com/repos/Appdynamics/ConfigMyApp/releases/latest \
            | grep "tag_name" \
            | awk '{print "https://github.com/Appdynamics/ConfigMyApp/archive/" substr(\$2, 2, length(\$2)-3) ".zip"}') \
            ; curl -L -o ConfigMyApp.zip \$LOCATION

            ls -ltr

            apt-get install unzip -y
            unzip ConfigMyApp.zip
            cd ConfigMyApp-* 

            #Branding 

            background="https://user-images.githubusercontent.com/2548160/88209855-36123d80-cc4b-11ea-8a46-8bf8bd02bbb5.jpg"
            curl -o "branding/background.jpg" \$background 
            logo="https://user-images.githubusercontent.com/2548160/88204707-9a310380-cc43-11ea-9149-561199f144af.png"
            curl -o "branding/logo.png" \$logo 

            if [ "\$CMA_BT_ONLY" = true ] || [ "\$CMA_CONFIGURE_BT" = true ]; then
              cp ../bt_config/${params.CMA_APPLICATION_NAME}-configBT.json bt_config/configBT.json
            fi

            ls -ltr

            echo "Start script"
            if [ "\$CMA_INCLUDE_DATABASE" = true ]; then 
              ./start.sh --controller-port=443 --include-database --database-name='${params.CMA_DATABASE_NAME}' --overwrite-health-rules --use-branding --logo-name="logo.png" --background-name="background.jpg"
            else
              ./start.sh --controller-port=443 --overwrite-health-rules --use-branding --logo-name="logo.png" --background-name="background.jpg"
            fi
            echo "End script"
          """
        }
      }
    } 
  }
}
