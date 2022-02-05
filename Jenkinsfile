pipeline {
	agent {label 'vagrant-worker'}
stages{
    stage ('Update machine'){
	 steps {
		sh 'sudo apt-get update'
	    }
	  }
	stage('Install ChefDK'){
	  steps{
			script{
				def chefDKexists  = fileExists '/usr/bin/chef-client'
				if (chefDKexists) {
					echo 'Skipping Chef install...already installed'
				}else{
					sh '''#!/bin/bash

				    		  wget wget https://packages.chef.io/files/stable/chefdk/4.13.3/ubuntu/18.04/chefdk_4.13.3-1_amd64.deb
				   		  sudo dpkg -i chefdk_4.13.3-1_amd64.deb
               				    '''
					/* sh 'wget wget https://packages.chef.io/files/stable/chefdk/4.13.3/ubuntu/18.04/chefdk_4.13.3-1_amd64.deb'
					   sh 'sudo dpkg -i chefdk_4.13.3-1_amd64.deb' */
				}
			}
		}
	}

		
	stage ('Creating directory for the configuration...'){
		steps{
			script{
		       	  def dirExists  = fileExists '/home/vagrant/chef-repo'
				  if (dirExists) {
					echo 'Skipping creating directory ...directory present'
				}else{
				    sh 'mkdir ~/chef-repo/ &&  mkdir ~/chef-repo/.chef'
				}			
			}
    	      }
    	}
	stage('Copy server credentials'){
		steps{
		withCredentials([file(credentialsId: 'chef-user-key', variable: 'USER'),
				 file(credentialsId: 'chef-org-key',  variable: 'ORG'),
				 file(credentialsId: 'chef-config-key', variable: 'CONFIG')]) {
			      sh '''
				    set +x
				    sudo cp --recursive "$USER"  ~/chef-repo/.chef/
						sudo cp --recursive "$ORG"  ~/chef-repo/.chef/
						sudo cp --recursive "$CONFIG"  ~/chef-repo/.chef/
						cd ~/chef-repo/.chef/

				 '''
		   }
	    }
	 }
	 stage('knife SSL certificates from the server'){
		steps{

		      sh '''
			    set +x
			    cd ~/chef-repo
			    sudo knife ssl fetch

			 '''
		}
	 }
	 stage('Bootstrap a Node'){
		steps{
		/* add all nodes you need */
	      sh '''
	    	 set +x
	   	 cd ~/chef-repo/.chef
	   	 knife bootstrap 192.168.1.70 -x vagrant -P vagrant --node-name test  --sudo -y

		 '''
		}
	 }
	 
 	 stage('Creating cookbooks directory...'){
	    steps{
		script{
		  def dirExists  = fileExists '/home/vagrant/chef-repo/cookbooks'
		  	if (dirExists) {
				echo 'Skipping creating directory ...directory present'
		        }else{
		        	sh 'mkdir ~/chef-repo/cookbooks'
		    }			
		}
	     }
	 }


	 stage('Clone github repo & download Cookbook'){
		steps{
			script{
				def repoCloned  = fileExists '/home/vagrant/jenkins-agent/workspace/chef-conf-pipeline/apache'
				    if (repoCloned){
					  sh '''
						echo 'Skipping clone repo ... repo cloned'
						mv /home/vagrant/jenkins-agent/workspace/chef-conf-pipeline/apache ~/chef-repo/cookbooks

					     '''	
				    }else{
					 sh 'git clone https://github.com/abdallauno1/apache.git' 					 
					 echo "$JOB_NAME" 
					    
				}	
				sh 'mv /$WORKSPACE/$JOB_NAME/apache'
		     }
		 }
	 }
	 stage('Upload the cookbook and add to the Node'){
		steps{
				/* add the cookbook in the node you can add all nodes */

			      sh '''
				    set +x
				    cd ~/chef-repo/cookbooks
				    knife cookbook upload apache
				    knife node run_list add test recipe[apache::default]

				 '''
		}
	   }

	  stage('Run the cookbook'){
		steps{

			      sh '''
		      	    set +x
		      	    cd ~/chef-repo/cookbooks
		      	    knife cookbook upload apache
		      	    knife node run_list add test recipe[apache::default]
    			withCredentials ([sshUserPrivateKey(credentialsId: 'vagrant-test', keyFileVariable: 'AGENT_SSHKEY', passphraseVariable: '',usernameVariable:'')]){

			  		sh "kinfe ssh 'role:webserver' -x vagrant -i $AGENT_SSHKEY 'sudo chef-client'"

			 } 
			 '''
		}
	     }


   }
  }
