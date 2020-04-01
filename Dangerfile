result_files = Dir.glob("rspec-junit/rspec-*")
# result_files = %w(rspec-junit/rspec-0 rspec-junit/rspec-1 rspec-junit/rspec-2 rspec-junit/rspec-3 rspec-junit/rspec-4 rspec-junit/rspec-5 rspec-junit/rspec-6)
#junit.parse_files result_files
#junit.show_skipped_tests = true
# junit.headers = %i(file name message)
#junit.report
# all_test = junit.tests.map(&:attributes)
# slowest_test = all_test.sort_by { |attributes| attributes[:time].to_f }.last
# message "#{slowest_test[:name]} took #{slowest_test[:time]} seconds"






def parse_files(*files)
  require 'ox'
  @tests = []
  failed_tests = []

  Array(files).flatten.each do |file|
    raise "No JUnit file was found at #{file}" unless File.exist? file

    xml_string = File.read(file)
    xml_string.gsub!(/\n/,"&#xA;")
    xml_string.sub("`","&apos;")

    xml_string.gsub!(/[‘’]/, "&apos;")

    message "#{xml_string}"
    doc = Ox.parse(xml_string)

    suite_root = doc.nodes.first.value == 'testsuites' ? doc.nodes.first : doc
    @tests += suite_root.nodes.map(&:nodes).flatten.select { |node| node.kind_of?(Ox::Element) && node.value == 'testcase' }

    failed_suites = suite_root.nodes.select { |suite| suite[:failures].to_i > 0 || suite[:errors].to_i > 0 }
    failed_tests += failed_suites.map(&:nodes).flatten.select { |node| node.kind_of?(Ox::Element) && node.value == 'testcase' }
  end

  @failures = failed_tests.select do |test|
    test.nodes.count > 0
  end.select do |test|
    node = test.nodes.first
    node.kind_of?(Ox::Element) && node.value == 'failure'
  end

  @errors = failed_tests.select do |test|
    test.nodes.count > 0
  end.select do |test|
    node = test.nodes.first
    node.kind_of?(Ox::Element) && node.value == 'error'
  end

  @skipped = @tests.select do |test|
    test.nodes.count > 0
  end.select do |test|
    node = test.nodes.first
    node.kind_of?(Ox::Element) && node.value == 'skipped'
  end

  @passes = @tests - @failures - @errors - @skipped
end

# Causes a build fail if there are test failures,
# and outputs a markdown table of the results.
#
# @return   [void]
# def report
#   return if failures.nil? # because danger calls `report` before loading a file
#   warn("Skipped #{skipped.count} tests.") if show_skipped_tests && skipped.count > 0

#   unless failures.empty? && errors.empty?
#     fail('Tests have failed, see below for more information.', sticky: false)
#     message = "### Tests: \n\n"

#     tests = (failures + errors)

#     common_attributes = tests.map{|test| test.attributes.keys }.inject(&:&)

#     # check the provided headers are available
#     unless headers.nil?
#       not_available_headers = headers.select { |header| not common_attributes.include?(header) }
#       raise "Some of headers provided aren't available in the JUnit report (#{not_available_headers})" unless not_available_headers.empty?
#     end

#     keys = headers || common_attributes
#     attributes = keys.map(&:to_s).map(&:capitalize)

#     # Create the headers
#     message << attributes.join(' | ') + "|\n"
#     message << attributes.map { |_| '---' }.join(' | ') + "|\n"

#     # Map out the keys to the tests
#     tests.each do |test|
#       row_values = keys.map { |key| test.attributes[key] }.map { |v| auto_link(v) }
#       message << row_values.join(' | ') + "|\n"
#     end

#     markdown message
#   end
# end

# def auto_link(value)
#   if File.exist?(value) && defined?(@dangerfile.github)
#     github.html_link(value, full_path: false)
#   else
#     value
#   end
# end



parse_files result_files

message @failures.to_s

@failures.each do |failure|
  msg = failure.locate("failure/*")
  fail(msg)
  failure.nodes.each do |node|
    node.nodes.each do |n|
      message "#{n}"
      # message "Value: #{n.value.to_s}"
    end
  end
end