require 'hpricot'
require 'hpricot_patches'

module HpricotRadiantTestHelper
  def hp_setup(input, url = nil)
    setup_behavior(url)
    @output = HpricotRadiantTestHelper::DocumentOutput.new(@behavior.render_text(input))  
  end
  
  # returns the inner content of
  # the first tag found by the css query
  def tag(css_query)
    @output.content_for(css_query)
  end
  
  # returns an array of tag contents
  # for all of the tags found by the
  # css query
  def tags(css_query)
    @output.content_for_all(css_query)
  end
  
  # returns a raw Hpricot::Elem object
  # for the first result found by the query
  def element(css_query)
    @output[css_query].first
  end
  
  # returns an array of Hpricot::Elem objects
  # for the results found by the query
  def elements(css_query)
    @output[css_query]
  end
  
  # small utility class for working with
  # the Hpricot parser class
  class DocumentOutput
    def initialize(response_body)
      @parser = Hpricot.parse(response_body)
    end

    def content_for(css_query)
      @parser.search(css_query).first.inner_text
    end

    def content_for_all(css_query)
      @parser.search(css_query).collect(&:inner_text)
    end

    def [](css_query)
      @parser.search(css_query)
    end
  end
  
  protected

    def setup_behavior(url)
      @behavior = @page.behavior
      @behavior.request = ActionController::TestRequest.new
      @behavior.request.request_uri = 'http://testhost.tld' + (url || @page.url)
      @behavior.response = ActionController::TestResponse.new
    end
end