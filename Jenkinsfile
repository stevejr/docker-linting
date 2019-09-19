def lintingRules
def stackYaml

stage("Checkout") {
  checkout scm
}

stage("test linter") {
    lintingRules = readJSON text: '''
{
    "id": "default_bundle",
    "policies": [
        {
            "comment": "Single policy for performing vulnerability checks, dockerfiles checks, and some container image best practice checks.", 
            "id": "48e6f7d6-1765-11e8-b5f9-8b6f228548b6", 
            "name": "Default Policy",
            "version": "1_0", 
            "rules": [
                {
                    "action": "STOP", 
                    "gate": "metadata", 
                    "params": [
                        {
                            "name": "attribute", 
                            "value": "deploy.resources.limits.cpus"
                        }, 
                        {
                            "name": "check", 
                            "value": "not_exists"
                        }
                    ], 
                    "trigger": "attribute"
                },
                {
                    "action": "WARN", 
                    "gate": "metadata", 
                    "params": [
                        {
                            "name": "attribute", 
                            "value": "deploy.resources.limits.cpus"
                        }, 
                        {
                            "name": "check", 
                            "value": ">"
                        }, 
                        {
                            "name": "value", 
                            "value": "1"
                        }
                    ], 
                    "trigger": "attribute"
                }
            ],
        }
    ],
}
'''

println lintingRules

stackYaml = readYaml file: sample-stack.yml
println stackYaml
}

stage('Lint') {

    for(svc in stackYaml.services) {
        println svc
        processLinting(svc)
    }
}

def processLinting(svc) {

    for(rule in lintingRules.policies.rules) {
        println rule
    }
}
