Wick-Infect
===========

Installs a Bash configuration at `/usr/local/lib/wick-infect` that shell scripts can source into their environment in order to get a copy of all of the library functions.

The file is first generated in the `depends` file of this formula.

Examples

      #!/usr/bin/env bash
      # This example is for a shell script that can run on the target machine
      # after configuring has been performed.

      # Source the library of functions
      . /usr/local/lib/wick-infect

      # Enable strict mode to be safe
      wickStrictMode

      # Now the script can download URLs
      wickGetUrl http://google.com/ google.html

      # Define a function
      callMyFunction() {
          local target verbose

          wickGetOption verbose verbose "$@"
          wickGetArgument target 0 "$@"

          if [[ ! -z "$verbose" ]]; then
              echo "Verbose mode enabled"
          fi

          # Can pass back values as well
          local "$target" && wickIndirect "$target" "data"
      }

      callMyFunction --verbose result
      echo "Result is: $result"

Returns nothing.


