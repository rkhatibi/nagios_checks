#!/usr/bin/env ruby
#vim:syntax=ruby

# A simple nagios check that should be run as root
# perhaps under the mcollective NRPE plugin and
# can check when the last run was done of puppet.
# It can also check fail counts and skip machines
# that are not enabled
#
# The script will use the puppet last_run-summar.yaml
# file to determine when last Puppet ran else the age
# of the statefile.

require 'yaml'

lockfile = '/var/lib/puppet/state/puppetdlock'
statefile = '/var/lib/puppet/state/state.yaml'
summaryfile = '/var/lib/puppet/state/last_run_summary.yaml'

running = false

lastrun_failed = false
failcount = nil 

if File.directory?(File.dirname(lockfile))
  first_run = true
  if File.exists?(lockfile)
    running = true
  end
else 
  first_run = false
end

if File.exist?(summaryfile)
  begin
    summary = YAML.load_file(summaryfile)

    # machines that outright failed to run like on missing dependencies
    # are treated as huge failures.  The yaml file will be valid but
    # it wont have anything but last_run in it
    unless summary.include?('events')
      failcount = 99
    else
      # and unless there are failures, the events hash just wont have the failure count
      failcount = summary['events']['failure'] || 0
    end
  rescue
    failcount = 0
    summary = nil
  end
end

if first_run == false
  puts 'CRITICAL: Puppet did not start'
  exit 2
elsif running == true
  puts 'OK: Puppet is still running'
  exit 0
elsif failcount == 0
  puts "OK: Puppet last run had #{failcount} failures"
  exit 0
elsif failcount >= 1
  puts "CRITICAL: Puppet last run had #{failcount} failures"
  exit 2
end
