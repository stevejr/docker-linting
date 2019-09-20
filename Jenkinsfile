def lintingRules
def stackYaml

node {
    stage("Checkout") {
      checkout scm
      sh """
        pwd
        ls -ltra
      """
    }
    
    stage("test linter") {
        lintingRules = readJSON file: 'sample-policy.json'
    
        println lintingRules
    
        stackYaml = readYaml file: 'sample-stack.yml'
        println stackYaml
    }
    
    stage('Lint') {
        def result = processServices(stackYaml, lintingRules)
        if (result.len > 0) {
            println "Overall validation of stack manifest failed: ${result}"
            currentBuild.result = 'FAILURE'
        }
    }
}

@NonCPS
def processServices(serviceList, policy) {
  def overallResult = []
  
  serviceList.services.each {svc ->
    println "Processing service: ${svc.key}"
    
    
  	def result = processLinting(svc, policy)
    overallResult.add(result)
  }
  
  return overallResult
}

@NonCPS
def processLinting(svc, policy) {
  def svcRuleResult = []
  def overallResult = true
  
  def shell = new GroovyShell( new Binding( [svc:svc].withDefault{ it } ) )

  policy.policies.rules.each { rule ->
    def ruleToAction = processRule(rule)
    def command = "if (svc.getAt(\"value\")?.${ruleToAction}) {return true} else {return false}"
    println "Rule that will be executed: $command"
    def myRes = shell.evaluate( command )
    //println "Rule evaluated to: $myRes"
    
    if (!myRes) {
      svcRuleResult.add(["svc": "${svc.key}", "rule": "${ruleToAction}", "result": "${myRes}"])
    }
  }
  
  svcRuleResult.each { res -> 
    println "Service ${res.svc} failed rule validation: \n   Failed Command: ${res.rule}"
    overallResult = false
  }
  return svcRuleResult
}

@NonCPS
def processRule(rule) {
    def attribute = ""
    def check = ""
    def value = ""
  
  	println "Processing rule: $rule"

    rule.params.each { param ->
        println "Processing param: $param"
        switch ("${param.name}") {
            case "attribute":
                attribute = param.value
                break
            case "check":
                check = param.value
                break
            case "value":
                value = param.value
                break
        }
    }      
      
    if (check.equals("exists")) {
        return "${attribute}"
    } else {
    	return "${attribute} ${check} ${value}"
    }
}
