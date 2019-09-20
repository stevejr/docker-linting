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

  stage('Lint') {
    lintingRules = readJSON file: 'sample-policy.json'
    println "DEBUG: Stage('Lint') - lintingRules: ${lintingRules}"

    stackYaml = readYaml file: 'sample-stack.yml'
    println "DEBUG: Stage('Lint') - stackYaml: ${stackYaml}"

    def result = processServices(stackYaml, lintingRules)
    if (result.size() > 0) {
      println "DEBUG: Stage('Lint') - Overall validation of stack manifest failed: ${result}"
      currentBuild.result = 'FAILURE'
    }
  }
}

@NonCPS
def processServices(serviceList, policy) {
  def overallResult = []
  
  serviceList.services.each {svc ->
    println "DEBUG: processServices - Processing service: ${svc.key}"
    
  	def result = processLinting(svc, policy)
    overallResult.add(result)
    println "DEBUG: processServices - overallResult contents: ${overallResult}"
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
    
//    def command = "if (svc.getAt(\"value\")?.${ruleToAction}) {return true} else {return false}"
//    println "DEBUG: processLinting - Rule that will be executed: $command"
    
//    def myRes = shell.evaluate( command )
//    println "DEBUG: processLinting - Rule that will be executed: $command"
    
//    def myRes = shell.evaluate( command )
//    println "DEBUG: processLinting - Rule evaluated to: $myRes"

//    println "DEBUG: processLinting - Rule evaluated to: $myRes"
    
//    if (!myRes) {
//      svcRuleResult.add(["svc": "${svc.key}", "rule": "${ruleToAction}", "result": "${myRes}"])
//    }

    if (!evalProperty(svc.getAt('value'), ruleToAction)) {
      svcRuleResult.add(["svc": "${svc.key}", "rule": "${ruleToAction}", "result": "Failed"])
    }
  }
  
  svcRuleResult.each { res -> 
    println "DEBUG: processLinting - Service ${res.svc} failed rule validation: \n   Failed Command: ${res.rule}"
    overallResult = false
  }
                                
  return svcRuleResult
}

@NonCPS
def processRule(rule) {
  def attribute = ""
  def check = ""
  def value = ""

  println "DEBUG: processRule - Processing rule: $rule"

  rule.params.each { param ->
    println "DEBUG: processRule - Processing param: $param"
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

@NonCPS
def evalProperty(object, String property) {
  Eval.x(object, 'x.' + property)
}
