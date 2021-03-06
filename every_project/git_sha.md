# Git SHA footer
For every project, please add version and deployment details to the footer. This should show the git branch that was deployed, the SHA code, and the time of server deploy (or "Not in deployed environment" when project is running locally).

Doing this enables easier troubleshooting because it's clear precisely what version of the software is installed, and gives bug reporters an unambiguous way to specify the version of the code against which they are reporting.

For example, here's an example of this footer in a Hyrax 1 application:

1. Set up `dotenv` and ensure you have an environment variable the references the name of your project.
1. in `config/initializers/git_sha.rb`:

  ```ruby
  revisions_logfile = "/opt/#{ENV['PROJECT_NAME']}/revisions.log"
  GIT_SHA =
    if Rails.env.production? && File.exist?(revisions_logfile)
      `tail -1 #{revisions_logfile}`.chomp.split(" ")[3].gsub(/\)$/, '')
    elsif Rails.env.development? || Rails.env.test?
      `git rev-parse HEAD`.chomp
    else
      "Unknown SHA"
    end

  BRANCH =
    if Rails.env.production? && File.exist?(revisions_logfile)
      `tail -1 #{revisions_logfile}`.chomp.split(" ")[1]
    elsif Rails.env.development? || Rails.env.test?
      `git rev-parse --abbrev-ref HEAD`.chomp
    else
      "Unknown branch"
    end

  LAST_DEPLOYED =
    if Rails.env.production? && File.exist?(revisions_logfile)
      deployed = `tail -1 #{revisions_logfile}`.chomp.split(" ")[7]
      Date.parse(deployed).strftime("%d %B %Y")
    else
      "Not in deployed environment"
    end
  ```
1. In `app/views/shared/_footer.html.erb`:

  ```html
  Version <span title="<%= GIT_SHA %>"><%= BRANCH %> updated <%= LAST_DEPLOYED %></span>
  ```
