.. The contents of this file are included in multiple topics.
.. This file should not be changed in a way that hinders its ability to appear in multiple documentation sets.

Use the ``control_group`` method to define an audit. Each ``control_group`` object contains one (or more) ``control`` blocks that define the tests within the audit. 

The syntax for the ``control_group`` method is as follows:

.. code-block:: ruby

   control_group "audit name" do
     control "name" do
       it "should do something" do
         expect(something).to be_something
     end
   end

where:

* ``control_group`` defines the block of code for the audit to be performed during the |chef client| run. The ``control_group`` block is part of the |chef| |dsl recipe|
* ``control`` defines each individual audit. Ideally, each ``control`` block maps to a specific audit. There is no limit to the number of ``control`` blocks that may defined within ``control_group``
* Each ``control`` block must define at least one validation to perform. Validations are contained within an ``it`` block. Each ``it`` block is processed as an individual statement during the |chef client| run's audit mode phase and is shown individually in the output
* An ``expect(something).to be_something`` represents a statement that defines each individual test. These statements are defined using patterns similar to |rspec| matchers. For example, ``to``, ``to_not``, ``be``, and so on. For example, a test that expects the |postgresql| pacakge to not be installed would be similar to ``expect(package("postgresql")).to_not be_installed``

For example:

.. code-block:: ruby

   control_group "Audit Mode" do
   
     control "mysql package" do
       it "should be installed" do
         expect(package("mysql")).to be_installed.with_version("5.6")
       end
     end
   
     control "postgres package" do
       it "should not be installed" do
         expect(package("postgresql")).to_not be_installed
       end
     end
   
     control "mysql service" do
       let(:mysql_service) { service("mysql") }
       it "should be enabled" do
         expect(mysql_service).to be_enabled
       end
       it "should be running" do
         expect(mysql_service).to be_running
       end
     end
   
     control "mysql config directory" do
       let(:config_dir) { file("/etc/mysql") }
       it "should exist with correct permissions" do
         expect(config_dir).to be_directory
         expect(config_dir).to be_mode(0700)
       end
       it "should be owned by the db user" do
         expect(config_dir).to be_owned_by('db_service_user')
       end
     end
   
     control "mysql config file" do
       let(:config_file) { file("/etc/mysql/my.cnf") }
       it "should exist with correct permissions" do
         expect(config_file).to be_file
         expect(config_file).to be_mode(0400)
       end
       it "should contain required configuration" do
         expect(config_file.content).to match(/default-time-zone='UTC'/)
       end
     end
   
   end

When the |chef client| runs the ``control_group`` block is processed during the audit phase.

If the audit was successful, the |chef client| will return output similar to:

.. code-block:: bash

   Audit Mode
     mysql package
       should be installed
     postgres package
       should not be installed
     mysql service
       should be enabled
       should be running
     mysql config directory
       should exist with correct permissions
       should be owned by the db user
     mysql config file
       should exist with correct permissions
       should contain required configuration


If an audit was unsuccessful, the |chef client| will return output similar to:

.. code-block:: bash

   Starting audit phase
   
   Audit Mode
     mysql package
     should be installed (FAILED - 1)
   postgres package
     should not be installed
   mysql service
     should be enabled (FAILED - 2)
     should be running (FAILED - 3)
   mysql config directory
     should exist with correct permissions (FAILED - 4)
     should be owned by the db user (FAILED - 5)
   mysql config file
     should exist with correct permissions (FAILED - 6)
     should contain required configuration (FAILED - 7)
   
   Failures:
   
   1) Audit Mode mysql package should be installed
     Failure/Error: expect(package("mysql")).to be_installed.with_version("5.6")
       expected Package "mysql" to be installed
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:22:in 'block (3 levels) in from_file'
   
   2) Audit Mode mysql service should be enabled
     Failure/Error: expect(mysql_service).to be_enabled
       expected Service "mysql" to be enabled
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:35:in 'block (3 levels) in from_file'
   
   3) Audit Mode mysql service should be running
      Failure/Error: expect(mysql_service).to be_running
       expected Service "mysql" to be running
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:38:in 'block (3 levels) in from_file'
   
   4) Audit Mode mysql config directory should exist with correct permissions
     Failure/Error: expect(config_dir).to be_directory
       expected `File "/etc/mysql".directory?` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:45:in 'block (3 levels) in from_file'
   
   5) Audit Mode mysql config directory should be owned by the db user
     Failure/Error: expect(config_dir).to be_owned_by('db_service_user')
       expected `File "/etc/mysql".owned_by?("db_service_user")` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:49:in 'block (3 levels) in from_file'
   
   6) Audit Mode mysql config file should exist with correct permissions
     Failure/Error: expect(config_file).to be_file
       expected `File "/etc/mysql/my.cnf".file?` to return true, got false
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:56:in 'block (3 levels) in from_file'
   
   7) Audit Mode mysql config file should contain required configuration
     Failure/Error: expect(config_file.content).to match(/default-time-zone='UTC'/)
       expected "-n\n" to match /default-time-zone='UTC'/
       Diff:
       @@ -1,2 +1,2 @@
       -/default-time-zone='UTC'/
       +-n
     # /var/chef/cache/cookbooks/grantmc/recipes/default.rb:60:in 'block (3 levels) in from_file'
   
   Finished in 0.5745 seconds (files took 0.46481 seconds to load)
   8 examples, 7 failures
   
   Failed examples:
   
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:21 # Audit Mode mysql package should be installed
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:34 # Audit Mode mysql service should be enabled
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:37 # Audit Mode mysql service should be running
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:44 # Audit Mode mysql config directory should exist with correct permissions
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:48 # Audit Mode mysql config directory should be owned by the db user
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:55 # Audit Mode mysql config file should exist with correct permissions
   rspec /var/chef/cache/cookbooks/grantmc/recipes/default.rb:59 # Audit Mode mysql config file should contain required configuration
   Auditing complete
