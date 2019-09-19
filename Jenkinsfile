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
        if (!result) {
            println "Overall validation of stack manifest failed"
        }
    }
}

@NonCPS
def processServices(serviceList, policy) {
  def overallResult = true
  
  serviceList.services.each {svc ->
    println "Processing service: ${svc.key}"
    
    
  	def result = processLinting(svc, policy)
    if (!result) {
      overallResult = false
    }
  }
  
  return overallResult
}

@NonCPS
def processLinting(svc, policy) {
  def svcRuleResult = []
  def overallResult = true
  
  def shell = new GroovyShell( new Binding( [svc:svc].withDefault{ it } ) )

  policy.policies.rules.each { rule ->
    def result = processRule(rule)
    def command = "if (svc.getAt(\"value\").${result}) {return true} else {return false}"
    println "Rule that will be executed: $command"
    //println svc.getAt('value').${result}
    def myRes = shell.evaluate( command )
    //println "Rule evaluated to: $myRes"
    
    if (!myRes) {
      svcRuleResult.add("${result}")
    }
  }
  
  if (svcRuleResult) {
    println "Service ${svc.key} failed rule validation: \n   Failed rules: ${svcRuleResult}"
    overallResult = false
  }
  
  return overallResult
}

@NonCPS
def processRule(rule) {
    def attribute = ""
    def check = ""
    def value = ""
  
  	// println "Processing rule: $rule"

    rule.params.each { param ->
        // println "Processing param: $param"
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
      
    if (check.equals("not-exists")) {
        return "!${attribute}"
    } else if (check.equals("exists")) {
        return "${attribute}"
    } else {
    	return "${attribute} ${check} ${value}"
    }
}
