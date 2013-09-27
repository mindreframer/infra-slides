## Outline:

- Slides

- mpx-ah-stats
  - downloading
  - parsing

- mpx-cli
  - ec2 list
  - ec2 refresh
  - ami list
  - ami refresh
  - sh/c
  - Packer bundling: sh/packer-wrapper packer-files/config/test-eu-west-1.json


- mpx-chef
  - cookbooks/scripts for live provisioning
  - same structure, as devbox



##### S3 Log Parser API

downloader = Mpx::Ah::LogDownloader.new
#clients = (1..3).to_a.map{|x| "client-#{x}"}
clients = (1..1).to_a.map{|x| "client-#{x}"}
clients.each do |client|
  downloader.download( "#{client}/2013-09-27")
  downloader.aggregate("#{client}/2013-09-27")
end


importer = Mpx::Ah::LogImporter.instance
res      = importer.import("working-files/summary/2013-09-25-client-2.log")

##### Mpx::Aws API
ec2 = Mpx::Aws::Connection.ec2


# changes the Name tag
Mpx::Aws::EC2.change_instance_name("i-78796e37", "mongo-server-1")

# find by name(-tag)
Mpx::Aws::EC2.first_by_name("some-server-2")

# list local EC2 records
Ec2Instance.all

# list local AMI records
AmiImage.all



# Create an AMI with Packer
sh/packer-wrapper packer-files/config/test-eu-west-1.json

# update the local state
$ mpx-cli ami refresh


# start a new instance

ubuntu_ami = "ami-8223c4f5"
instance = Mpx::Aws::Connection.ec2.instances.create(
  :image_id        => ubuntu_ami,
  :count           => 1,
  :key_name        => "mpx-testing",
  :security_groups => ["salt-minion", "salt-master"],
  :instance_type   => "m1.small",
)
Mpx::Aws::EC2.change_instance_name(instance.instance_id, "app-server-1")
sleep 1 while instance.status == :pending
Ec2Instance.refresh!


# login in to new server and runchef
$ mpx-cli login app-server-1