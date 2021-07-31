@Library('jenkins-shared-library@main') _
pipeline {
    properties([
    parameters([
        string(
            name: 'NUMBER',
            defaultValue: 1,
            description: 'just any number will do'
            ),
        ])
    )
    def number = params.NUMBER
    helloworld {
        NUMBER = number
    }
}
