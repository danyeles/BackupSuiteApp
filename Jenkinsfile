pipeline {
    agent any

    parameters {
        booleanParam(
            name: 'BACKUP_ARR_APPS',
            defaultValue: true,
            description: 'Backup Arr applications'
        )

        booleanParam(
            name: 'BACKUP_JELLYFIN',
            defaultValue: true,
            description: 'Backup Jellyfin configuration'
        )

        booleanParam(
            name: 'BACKUP_JENKINS',
            defaultValue: true,
            description: 'Backup Jenkins Home'
        )
    }

    environment {
        BACKUP_ROOT = '/home/docker/backups'
        DATESTAMP = "${new Date().format('yyyyMMdd-HHmm')}"
    }

    stages {

        stage('Create Backup Folders') {
            steps {
                sh '''
                    mkdir -p ${BACKUP_ROOT}/configs
                    mkdir -p ${BACKUP_ROOT}/docker-inspect
                '''
            }
        }

        stage('Backup ARR Applications') {
            when {
                expression { params.BACKUP_ARR_APPS }
            }

            steps {
                sh '''
                    APPS="
                    sonarr
                    radarr
                    prowlarr
                    readarr
                    lidarr
                    bazarr
                    qbittorrent
                    kavita
                    audiobookshelf
                    homepage
                    "

                    for APP in $APPS
                    do
                        if [ -d /home/docker/${APP}/config ]; then
                            echo "Backing up ${APP}"

                            tar -czf \
                            ${BACKUP_ROOT}/configs/${APP}-${DATESTAMP}.tar.gz \
                            /home/docker/${APP}/config
                        fi
                    done
                '''
            }
        }

        stage('Backup Jellyfin') {
            when {
                expression { params.BACKUP_JELLYFIN }
            }

            steps {
                sh '''
                    if [ -d /home/docker/jellyfin/config ]; then
                        tar -czf \
                        ${BACKUP_ROOT}/configs/jellyfin-${DATESTAMP}.tar.gz \
                        /home/docker/jellyfin/config
                    fi
                '''
            }
        }

        stage('Backup Jenkins') {
            when {
                expression { params.BACKUP_JENKINS }
            }

            steps {
                sh '''
                    if [ -d /var/jenkins_home ]; then
                        tar -czf \
                        ${BACKUP_ROOT}/configs/jenkins-${DATESTAMP}.tar.gz \
                        /var/jenkins_home
                    fi
                '''
            }
        }

        stage('Docker Inspect Export') {
            steps {
                sh '''
                    docker ps -a --format "{{.Names}}" | while read container
                    do
                        docker inspect ${container} > \
                        ${BACKUP_ROOT}/docker-inspect/${container}-${DATESTAMP}.json
                    done
                '''
            }
        }

        stage('Cleanup Old Backups') {
            steps {
                sh '''
                    find ${BACKUP_ROOT} \
                    -type f \
                    -mtime +30 \
                    -delete
                '''
            }
        }
    }

    post {
        success {
            echo "Backup completed successfully."
        }

        failure {
            echo "Backup failed."
        }
    }
}
