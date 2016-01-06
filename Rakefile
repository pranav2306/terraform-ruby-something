require "erb"
require "symmetric_encryption"

def check_file_existence(filename)
  print "#{filename} => "

  if File.exist?(filename)
    puts "exist."
  else
    puts "not exist!"
    abort
  end
end

namespace :terraform do
  desc "install terraform to this directory"
  task :install do
    terraform_version = File.read(".terraform_version").chomp
    download_url      = "https://releases.hashicorp.com/terraform/#{terraform_version}/terraform_#{terraform_version}_#{`uname`.match(/darwin/i) ? "darwin" : "linux"}_amd64.zip"

    system "curl -L #{download_url} -o terraform.zip"
    system "unzip terraform.zip -d vendor/"
    system "rm -f terraform.zip"
  end

  desc "check required files to exist, initialize cipher"
  task :init do
    check_file_existence "vendor/terraform"
    check_file_existence ".envrc"
    check_file_existence ENV["PRIVATE_KEY_FILE"]

    SymmetricEncryption.cipher = SymmetricEncryption::Cipher.new(
      key:         File.read(ENV["PRIVATE_KEY_FILE"]),
      cipher_name: "aes-256-cbc",
    )
  end

  desc "build *.tf.erb to .tf"
  task :build do
    Dir.glob("*.tf.erb").each do |path|
      File.open path.sub(/\.erb\z/, ""), "w" do |file|
        file.write ERB.new(File.read(path)).result
      end
    end
  end

  desc "decrypt terraform.tfstate.encrypted to terraform.tfstate"
  task :decrypt_state do
    Rake::Task["terraform:init"].invoke

    if File.exist?("terraform.tfstate.encrypted")
      puts "\ndecrypt terraform.tfstate.encrypted => terraform.tfstate"

      File.open("terraform.tfstate", "w") do |plain_file|
        File.open("terraform.tfstate.encrypted") do |encrypted_file|
          decrypted = SymmetricEncryption.decrypt(encrypted_file.read)
          plain_file.write decrypted
        end
      end
    end
  end

  desc "encrypt terraform.tfstate to terraform.tfstate.encrypted"
  task :encrypt_state do
    Rake::Task["terraform:init"].invoke
    puts "\nencrypt: terraform.tfstate => terraform.tfstate.encrypted"

    File.open("terraform.tfstate.encrypted", "w") do |encrypted_file|
      File.open("terraform.tfstate") do |plain_file|
        encrypted = SymmetricEncryption.encrypt(plain_file.read, false, true)
        encrypted_file.write encrypted
      end
    end
  end

  desc "terraform plan"
  task :plan do
    system "vendor/terraform plan"
    Rake::Task["terraform:encrypt_state"].invoke
  end

  desc "terraform apply"
  task :apply do
    system "vendor/terraform apply"
    Rake::Task["terraform:encrypt_state"].invoke
  end

  task :plan  => :decrypt_state
  task :plan  => :build
  task :apply => :decrypt_state
  task :apply => :build
end

task default: ["terraform:plan"]
