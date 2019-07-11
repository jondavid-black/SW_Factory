// For this pipeline to work, many of these commands must run as root.
//  The best way I see to do this is to run Jenkins under a Jenkins user.
//  Then setup that Jenkins user with special permissions in the sudoers.d directory for the needed commands.
//  If done correctly, this should allow the commands to be run by the Jenkins user without a password.

// But, I don't have time to work all that out so I'll just run Jenkins as root.  Do not do this!!

pipeline {
    agent  any 

    parameters {
        choice(
            choices: ['1', '2', '3'],
            description: 'Choose the server number to build.',
            name: 'SVR_NUM'
        )
    }

    stages {
        
        stage('Check for SD Card') {
            steps {
                sh 'lsblk -b /dev/sdc'
            }
        }

        stage('Pull image from artifact repo') {
            steps {
                // This is a total hack.  This should be a pull from a real artifact repo...which I haven't setup yet.
                sh 'mkdir -p ./images/raspbian-stretch-lite'
                sh 'cp /home/jd/Documents/DevOps/demo/SW_Factory/images/raspbian-stretch-lite/2019-04-08-raspbian-stretch-lite.img ./images/raspbian-stretch-lite'
            }
        }

        stage('Write Base Image to SD') {
            steps {
                sh 'dd bs=4M if=./images/raspbian-stretch-lite/2019-04-08-raspbian-stretch-lite.img of=/dev/sdc conv=fsync'
            }
        }

        stage('Mount SD Card') {
            steps {
                sh 'mkdir -p /media/sw_factory/boot'
                sh 'mount -t vfat /dev/sdc1 /media/sw_factory/boot'

                sh 'mkdir -p /media/sw_factory/rootfs'
                sh 'mount /dev/sdc2 /media/sw_factory/rootfs'
            }
        }

        stage('Write Bridge Network Config') {
            when {
                // Only write raspbian bridge config for svr 3
                expression { params.SVR_NUM =='1' }
            }
            steps {
                // This is another hack to allow me to post to GitHub and still protect wifi password confidentiallity.
                sh 'cp /home/jd/Documents/DevOps/demo/SW_Factory/config/rpi1/wpa_supplicant.conf_secret ./config/rpi1/wpa_supplicant.conf'

                // enable SSH on boot
                sh 'touch /media/sw_factory/boot/ssh'
                // setup wifi interface for DHCP on specific SSIDs
                sh 'cp ./config/rpi1/wpa_supplicant.conf /media/sw_factory/boot/wpa_supplicant.conf'
                // setup /etc/network/interfaces for static IP on eth0
                sh 'cp ./config/rpi1/dhcpcd.conf /media/sw_factory/rootfs/etc/dhcpcd.conf'
            }

        }
        stage('Write Worker Network Config') {
            parallel {
                stage('Configure RPI2 Network') {
                    when {
                        expression { params.SVR_NUM =='2' }
                    }
                    steps {
                        // enable SSH on boot
                        sh 'touch /media/sw_factory/boot/ssh'
                        // setup /etc/network/interfaces for DHCP on eth0
                        sh 'cp ./config/rpi2/dhcpcd.conf /media/sw_factory/rootfs/etc/dhcpcd.conf'
                    }
                }
                stage('Configure RPI3 Network') {
                    when {
                        expression { params.SVR_NUM =='3' }
                    }
                    steps {
                        // enable SSH on boot
                        sh 'touch /media/jd/boot/ssh'
                        // setup /etc/network/interfaces for DHCP on eth0
                        sh 'cp ./config/rpi3/dhcpcd.conf /media/sw_factory/rootfs/etc/dhcpcd.conf'
                    }
                }
            }
        }
        
        stage('Unmount SD Card') {
            steps {
                sh 'umount /media/sw_factory/boot'
                sh 'umount /media/sw_factory/rootfs'
                sh 'rm -rf /media/sw_factory'
            }
           
        }
    }
}