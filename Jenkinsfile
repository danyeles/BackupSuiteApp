pipeline {
    agent any

    environment {
        BACKUP_ROOT = '/mnt/Backups'
        DATESTAMP = sh(
            script: 'date +%Y%m%d-%H%M%S',
            returnStdout: true
        ).trim()
    }

    stages {

        stage('Create Backup Structure') {
            steps {
                sh '''
                    mkdir -p ${BACKUP_ROOT}/apps
                    mkdir -p ${BACKUP_ROOT}/docker-inspect
                '''
            }
        }

        stage('Backup Application Configs') {
            steps {

                script {

                    def backups = [
                        [
                            name: 'sonarr',
                            path: '/home/docker/sonarr/config'
                        ],
                        [
                            name: 'radarr',
                            path: '/home/docker/radarr/config'
                        ],
                        [
                            name: 'prowlarr',
                            path: '/home/docker/prowlarr/config'
                        ],
                        [
                            name: 'readarr',
                            path: '/home/docker/readarr/config'
                        ],
                        [
                            name: 'lidarr',
                            path: '/home/docker/lidarr/config'
                        ],
                        [
                            name: 'bazarr',
                            path: '/home/docker/bazarr/config'
                        ],
                        [
                            name: 'seerr',
                            path: '/home/docker/seerr/config'
                        ],
                        [
                            name: 'homepage',
                            path: '/home/docker/homepage/config'
                        ],
                        [
                            name: 'sortarr',
                            path: '/home/docker/sortarr/config'
                        ],
                        [
                            name: 'qbittorrent',
                            path: '/opt/stacks/qbittorrent/config'
                        ],
                        [
                            name: 'audiobookshelf-config',
                            path: '/mnt/Media/audiobookshelf/config'
                        ],
                        [
                            name: 'audiobookshelf-metadata',
                            path: '/mnt/Media/audiobookshelf/metadata'
                        ],
                        [
                            name: 'jellyfin-data',
                            path: '/var/lib/jellyfin/data'
                        ],
                        [
                            name: 'jellyfin-plugins',
                            path: '/var/lib/jellyfin/plugins'
                        ],
                        [
                            name: 'jellyfin-config',
                            path: '/etc/jellyfin'
                        ],
                        [
                            name: 'jellyfin-full',
                            path: '/var/lib/jellyfin'
                        ]
                        [
                            name: 'jellyfin-config',
                            path: '/etc/jellyfin'
                        ]
                    ]

                    backups.each { item ->

                        sh """
                            if [ -d "${item.path}" ]; then

                                mkdir -p ${BACKUP_ROOT}/apps/${item.name}

                                tar -czf \
                                ${BACKUP_ROOT}/apps/${item.name}/${item.name}-${DATESTAMP}.tar.gz \
                                ${item.path}

                                cp \
                                ${BACKUP_ROOT}/apps/${item.name}/${item.name}-${DATESTAMP}.tar.gz \
                                ${BACKUP_ROOT}/apps/${item.name}/latest.tar.gz

                                echo "Backed up ${item.name}"

                            else

                                echo "WARNING: ${item.path} not found"

                            fi
                        """
                    }
                }
            }
        }

        stage('Export Docker Container Definitions') {
            steps {

                sh '''
                    docker ps -a --format "{{.Names}}" | while read container
                    do
                        docker inspect "${container}" > \
                        ${BACKUP_ROOT}/docker-inspect/${container}.json
                    done
                '''
            }
        }

        stage('Create Manifest') {
            steps {

                sh '''
                    find ${BACKUP_ROOT} -type f \
                    | sort \
                    > ${BACKUP_ROOT}/backup-manifest.txt
                '''
            }
        }

        stage('Retention') {
            steps {

                sh '''
                    find ${BACKUP_ROOT}/apps \
                    -name "*.tar.gz" \
                    ! -name "latest.tar.gz" \
                    -mtime +30 \
                    -delete || true
                '''
            }
        }
    }

    post {

        success {
            echo '''
=========================================================
MEDIA STACK BACKUP COMPLETED
=========================================================
'''
        }

        failure {
            echo '''
=========================================================
MEDIA STACK BACKUP FAILED
=========================================================
'''
        }
    }
}
