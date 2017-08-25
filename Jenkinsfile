pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh '''
                    GIT_COMMIT=$(git rev-parse HEAD)
                    DEST=$(git show --pretty=format: --name-only ${GIT_COMMIT})
                    [[ ${BRANCH_NAME} == 'testing' ]] && REPO_NAME=${REPO_NAME}-${BRANCH_NAME}
                    for f in ${DEST[@]};do
                        if [[ $f == */PKGBUILD ]];then
                            PACKAGE=${f%/PKGBUILD}
                            source $f
                            POOL_DIR=${POOL_DIR}/${ARCH}/${POOL_NAME}
                            REPO_DIR=${REPO_DIR}/${REPO_NAME}
                            [[ $arch == 'any' ]] && ARCH='any'
                            VERSION=$pkgver
                            RELEASE=$pkgrel
                            DEPLOY=(${pkgname[@]})
                            buildpkg -p ${PACKAGE} -cu -z ${REPO_NAME}
                        fi
                    done
                '''
            }
            post {
                success {
                    sh '''
                        for pkg in ${DEPLOY[@]};do
                            FILE=${pkg}-${VERSION}-${RELEASE}-${ARCH}.${EXT}
                            signfile ${POOL_DIR}/${FILE}
                            deploypkg -x -p ${FILE} -r ${REPO_NAME}
                        done
                    '''
                }
            }
        }
    }
    environment {
        GIT_COMMIT = ''
        DEST = ''
        PACKAGE = ''
        POOL_NAME = 'world'
        REPO_NAME = 'world'
        ARCH = 'x86_64'
        VERSION = ''
        RELEASE = ''
        DEPLOY = ''
        EXT = 'pkg.tar.xz'
        POOL_DIR="${JENKINS_HOME}" + '/artools-workspace/pkg'
        REPO_DIR="${JENKINS_HOME}" + '/artools-workspace/repos'
    }
}
